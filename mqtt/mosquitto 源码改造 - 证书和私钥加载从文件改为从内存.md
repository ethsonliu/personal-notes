首先找到设置证书和私钥的位置，下面这个接口，位置在：`libs/options.c`，

```c++
/*
 * Function: mosquitto_tls_set
 *
 * Configure the client for certificate based SSL/TLS support. Must be called
 * before <mosquitto_connect>.
 *
 * Cannot be used in conjunction with <mosquitto_tls_psk_set>.
 *
 * Define the Certificate Authority certificates to be trusted (ie. the server
 * certificate must be signed with one of these certificates) using cafile.
 *
 * If the server you are connecting to requires clients to provide a
 * certificate, define certfile and keyfile with your client certificate and
 * private key. If your private key is encrypted, provide a password callback
 * function or you will have to enter the password at the command line.
 *
 * Parameters:
 *  mosq -        a valid mosquitto instance.
 *  cafile -      path to a file containing the PEM encoded trusted CA
 *                certificate files. Either cafile or capath must not be NULL.
 *  capath -      path to a directory containing the PEM encoded trusted CA
 *                certificate files. See mosquitto.conf for more details on
 *                configuring this directory. Either cafile or capath must not
 *                be NULL.
 *  certfile -    path to a file containing the PEM encoded certificate file
 *                for this client. If NULL, keyfile must also be NULL and no
 *                client certificate will be used.
 *  keyfile -     path to a file containing the PEM encoded private key for
 *                this client. If NULL, certfile must also be NULL and no
 *                client certificate will be used.
 *  pw_callback - if keyfile is encrypted, set pw_callback to allow your client
 *                to pass the correct password for decryption. If set to NULL,
 *                the password must be entered on the command line.
 *                Your callback must write the password into "buf", which is
 *                "size" bytes long. The return value must be the length of the
 *                password. "userdata" will be set to the calling mosquitto
 *                instance. The mosquitto userdata member variable can be
 *                retrieved using <mosquitto_userdata>.
 *
 * Returns:
 *	MOSQ_ERR_SUCCESS - on success.
 * 	MOSQ_ERR_INVAL -   if the input parameters were invalid.
 * 	MOSQ_ERR_NOMEM -   if an out of memory condition occurred.
 *
 * See Also:
 *	<mosquitto_tls_opts_set>, <mosquitto_tls_psk_set>,
 *	<mosquitto_tls_insecure_set>, <mosquitto_userdata>
 */
libmosq_EXPORT int mosquitto_tls_set(struct mosquitto *mosq,
		const char *cafile, const char *capath,
		const char *certfile, const char *keyfile,
		int (*pw_callback)(char *buf, int size, int rwflag, void *userdata))
{
    // 查看文件是否能够打开，若能打开，做下面的...
    
    mosq->tls_cafile      = mosquitto__strdup(cafile);
    mosq->tls_capath      = mosquitto__strdup(capath);
    mosq->tls_certfile    = mosquitto__strdup(certfile);
    mosq->tls_keyfile     = mosquitto__strdup(keyfile);
    mosq->tls_pw_callback = pw_callback;
}
```

这个接口只是简单的做些文件判断存不存在，如果存在就把文件路径赋给 mosq 实体。

接下来具体在哪里处理这些文件的呢？

`lib/net_mosq.c`中的`static int net__init_ssl_ctx(struct mosquitto *mosq)`中实现了主要 SSL 处理，

改造后的代码为（都打了中文注释）：

```c++
static int net__init_ssl_ctx(struct mosquitto *mosq)
{
	int ret;
	ENGINE *engine = NULL;
	uint8_t tls_alpn_wire[256];
	uint8_t tls_alpn_len;

	if(mosq->ssl_ctx){
		if(!mosq->ssl_ctx_defaults){
			return MOSQ_ERR_SUCCESS;
		}else if(!mosq->tls_cafile && !mosq->tls_capath && !mosq->tls_psk){
			log__printf(mosq, MOSQ_LOG_ERR, "Error: MOSQ_OPT_SSL_CTX_WITH_DEFAULTS used without specifying cafile, capath or psk.");
			return MOSQ_ERR_INVAL;
		}
	}

	/* Apply default SSL_CTX settings. This is only used if MOSQ_OPT_SSL_CTX
	 * has not been set, or if both of MOSQ_OPT_SSL_CTX and
	 * MOSQ_OPT_SSL_CTX_WITH_DEFAULTS are set. */
	if(mosq->tls_cafile || mosq->tls_capath || mosq->tls_psk){
		if(!mosq->ssl_ctx){
#if OPENSSL_VERSION_NUMBER < 0x10100000L
			mosq->ssl_ctx = SSL_CTX_new(SSLv23_client_method()); // 申请SSL会话环境
#else
			mosq->ssl_ctx = SSL_CTX_new(TLS_client_method());
#endif

			if(!mosq->ssl_ctx){
				log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to create TLS context.");
				COMPAT_CLOSE(mosq->sock);
				mosq->sock = INVALID_SOCKET;
				net__print_ssl_error(mosq);
				return MOSQ_ERR_TLS;
			}
		}

		if(!mosq->tls_version){
			SSL_CTX_set_options(mosq->ssl_ctx, SSL_OP_NO_SSLv3 | SSL_OP_NO_TLSv1);
#ifdef SSL_OP_NO_TLSv1_3
		}else if(!strcmp(mosq->tls_version, "tlsv1.3")){
			SSL_CTX_set_options(mosq->ssl_ctx, SSL_OP_NO_SSLv3 | SSL_OP_NO_TLSv1 | SSL_OP_NO_TLSv1_1 | SSL_OP_NO_TLSv1_2);
		}else if(!strcmp(mosq->tls_version, "tlsv1.2")){
			SSL_CTX_set_options(mosq->ssl_ctx, SSL_OP_NO_SSLv3 | SSL_OP_NO_TLSv1 | SSL_OP_NO_TLSv1_1 | SSL_OP_NO_TLSv1_3);
		}else if(!strcmp(mosq->tls_version, "tlsv1.1")){
			SSL_CTX_set_options(mosq->ssl_ctx, SSL_OP_NO_SSLv3 | SSL_OP_NO_TLSv1 | SSL_OP_NO_TLSv1_2 | SSL_OP_NO_TLSv1_3);
#else
		}else if(!strcmp(mosq->tls_version, "tlsv1.2")){
			SSL_CTX_set_options(mosq->ssl_ctx, SSL_OP_NO_SSLv3 | SSL_OP_NO_TLSv1 | SSL_OP_NO_TLSv1_1);
		}else if(!strcmp(mosq->tls_version, "tlsv1.1")){
			SSL_CTX_set_options(mosq->ssl_ctx, SSL_OP_NO_SSLv3 | SSL_OP_NO_TLSv1 | SSL_OP_NO_TLSv1_2);
#endif
		}else{
			log__printf(mosq, MOSQ_LOG_ERR, "Error: Protocol %s not supported.", mosq->tls_version);
			COMPAT_CLOSE(mosq->sock);
			mosq->sock = INVALID_SOCKET;
			return MOSQ_ERR_INVAL;
		}

		/* Disable compression */
		SSL_CTX_set_options(mosq->ssl_ctx, SSL_OP_NO_COMPRESSION);

		/* Set ALPN */
		if(mosq->tls_alpn) {
			tls_alpn_len = (uint8_t) strnlen(mosq->tls_alpn, 254);
			tls_alpn_wire[0] = tls_alpn_len;  // first byte is length of string
			memcpy(tls_alpn_wire + 1, mosq->tls_alpn, tls_alpn_len);
			SSL_CTX_set_alpn_protos(mosq->ssl_ctx, tls_alpn_wire, tls_alpn_len + 1);
		}

#ifdef SSL_MODE_RELEASE_BUFFERS
			/* Use even less memory per SSL connection. */
			SSL_CTX_set_mode(mosq->ssl_ctx, SSL_MODE_RELEASE_BUFFERS);
#endif

#if !defined(OPENSSL_NO_ENGINE)
		if(mosq->tls_engine){
			engine = ENGINE_by_id(mosq->tls_engine);
			if(!engine){
				log__printf(mosq, MOSQ_LOG_ERR, "Error loading %s engine\n", mosq->tls_engine);
				COMPAT_CLOSE(mosq->sock);
				mosq->sock = INVALID_SOCKET;
				return MOSQ_ERR_TLS;
			}
			if(!ENGINE_init(engine)){
				log__printf(mosq, MOSQ_LOG_ERR, "Failed engine initialisation\n");
				ENGINE_free(engine);
				COMPAT_CLOSE(mosq->sock);
				mosq->sock = INVALID_SOCKET;
				return MOSQ_ERR_TLS;
			}
			ENGINE_set_default(engine, ENGINE_METHOD_ALL);
			ENGINE_free(engine); /* release the structural reference from ENGINE_by_id() */
		}
#endif

		if(mosq->tls_ciphers){
			ret = SSL_CTX_set_cipher_list(mosq->ssl_ctx, mosq->tls_ciphers);
			if(ret == 0){
				log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to set TLS ciphers. Check cipher list \"%s\".", mosq->tls_ciphers);
#if !defined(OPENSSL_NO_ENGINE)
				ENGINE_FINISH(engine);
#endif
				COMPAT_CLOSE(mosq->sock);
				mosq->sock = INVALID_SOCKET;
				net__print_ssl_error(mosq);
				return MOSQ_ERR_TLS;
			}
		}
		if(mosq->tls_cafile || mosq->tls_capath){
			ret = SSL_CTX_load_verify_locations(mosq->ssl_ctx, mosq->tls_cafile, mosq->tls_capath); // 验证和加载服务端根证书
			if(ret == 0){
#ifdef WITH_BROKER
				if(mosq->tls_cafile && mosq->tls_capath){
					log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to load CA certificates, check bridge_cafile \"%s\" and bridge_capath \"%s\".", mosq->tls_cafile, mosq->tls_capath);
				}else if(mosq->tls_cafile){
					log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to load CA certificates, check bridge_cafile \"%s\".", mosq->tls_cafile);
				}else{
					log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to load CA certificates, check bridge_capath \"%s\".", mosq->tls_capath);
				}
#else
				if(mosq->tls_cafile && mosq->tls_capath){
					log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to load CA certificates, check cafile \"%s\" and capath \"%s\".", mosq->tls_cafile, mosq->tls_capath);
				}else if(mosq->tls_cafile){
					log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to load CA certificates, check cafile \"%s\".", mosq->tls_cafile);
				}else{
					log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to load CA certificates, check capath \"%s\".", mosq->tls_capath);
				}
#endif
#if !defined(OPENSSL_NO_ENGINE)
				ENGINE_FINISH(engine);
#endif
				COMPAT_CLOSE(mosq->sock);
				mosq->sock = INVALID_SOCKET;
				net__print_ssl_error(mosq);
				return MOSQ_ERR_TLS;
			}
			if(mosq->tls_cert_reqs == 0){
				SSL_CTX_set_verify(mosq->ssl_ctx, SSL_VERIFY_NONE, NULL); // 服务端证书验证
			}else{
				SSL_CTX_set_verify(mosq->ssl_ctx, SSL_VERIFY_PEER, mosquitto__server_certificate_verify);
			}

			if(mosq->tls_pw_callback){
				SSL_CTX_set_default_passwd_cb(mosq->ssl_ctx, mosq->tls_pw_callback);
				SSL_CTX_set_default_passwd_cb_userdata(mosq->ssl_ctx, mosq);
			}

			if(mosq->tls_certfile)
			{
				//ret = SSL_CTX_use_certificate_chain_file(mosq->ssl_ctx, mosq->tls_certfile); // 客户端证书
				BIO  *tls_cert_bio = BIO_new_mem_buf((void*)mosq->tls_certfile, -1);
				if (tls_cert_bio == NULL)
					printf("[mosquitto] net__init_ssl_ctx - BIO_new_mem_buf(tls_certfile) return NULL\n");

				X509 *tls_cert_x509 = PEM_read_bio_X509(tls_cert_bio, NULL, 0, NULL);
				if (tls_cert_x509 == NULL)
					printf("[mosquitto] net__init_ssl_ctx - PEM_read_bio_X509 return NULL\n");

				ret = SSL_CTX_use_certificate(mosq->ssl_ctx, tls_cert_x509);

				X509_free(tls_cert_x509);
				BIO_set_close(tls_cert_bio, BIO_NOCLOSE);
				BIO_free(tls_cert_bio);

				if(ret != 1){
#ifdef WITH_BROKER
					log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to load client certificate, check bridge_certfile \"%s\".", mosq->tls_certfile);
#else
					log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to load client certificate \"%s\".", mosq->tls_certfile);
#endif
#if !defined(OPENSSL_NO_ENGINE)
					ENGINE_FINISH(engine);
#endif
					COMPAT_CLOSE(mosq->sock);
					mosq->sock = INVALID_SOCKET;
					net__print_ssl_error(mosq);
					return MOSQ_ERR_TLS;
				}
			}

			if(mosq->tls_keyfile)
			{
				if(mosq->tls_keyform == mosq_k_engine) // 如果没有调用 mosquitto_string_option 设置过，这里就不用管
				{
#if !defined(OPENSSL_NO_ENGINE)
					UI_METHOD *ui_method = net__get_ui_method();
					if(mosq->tls_engine_kpass_sha1){
						if(!ENGINE_ctrl_cmd(engine, ENGINE_SECRET_MODE, ENGINE_SECRET_MODE_SHA, NULL, NULL, 0)){
							log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to set engine secret mode sha1");
							ENGINE_FINISH(engine);
							COMPAT_CLOSE(mosq->sock);
							mosq->sock = INVALID_SOCKET;
							net__print_ssl_error(mosq);
							return MOSQ_ERR_TLS;
						}
						if(!ENGINE_ctrl_cmd(engine, ENGINE_PIN, 0, mosq->tls_engine_kpass_sha1, NULL, 0)){
							log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to set engine pin");
							ENGINE_FINISH(engine);
							COMPAT_CLOSE(mosq->sock);
							mosq->sock = INVALID_SOCKET;
							net__print_ssl_error(mosq);
							return MOSQ_ERR_TLS;
						}
						ui_method = NULL;
					}

					EVP_PKEY *pkey = ENGINE_load_private_key(engine, mosq->tls_keyfile, ui_method, NULL);
					if(!pkey){
						log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to load engine private key file \"%s\".", mosq->tls_keyfile);
						ENGINE_FINISH(engine);
						COMPAT_CLOSE(mosq->sock);
						mosq->sock = INVALID_SOCKET;
						net__print_ssl_error(mosq);
						return MOSQ_ERR_TLS;
					}

					if(SSL_CTX_use_PrivateKey(mosq->ssl_ctx, pkey) <= 0){
						log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to use engine private key file \"%s\".", mosq->tls_keyfile);
						ENGINE_FINISH(engine);
						COMPAT_CLOSE(mosq->sock);
						mosq->sock = INVALID_SOCKET;
						net__print_ssl_error(mosq);
						return MOSQ_ERR_TLS;
					}
#endif
				}
				else // mosq->tls_keyform == mosq_k_pem
				{
					//ret = SSL_CTX_use_PrivateKey_file(mosq->ssl_ctx, mosq->tls_keyfile, SSL_FILETYPE_PEM); // 使用客户端私钥文件
					BIO  *tls_key_bio = BIO_new_mem_buf((void*)mosq->tls_keyfile, -1);
					if (tls_key_bio == NULL)
					printf("[mosquitto] net__init_ssl_ctx - BIO_new_mem_buf(tls_keyfile) return NULL\n");

					RSA  *tls_rsa_key = PEM_read_bio_RSAPrivateKey(tls_key_bio, NULL, 0, NULL);
					if (tls_rsa_key == NULL)
					printf("[mosquitto] net__init_ssl_ctx - PEM_read_bio_RSAPrivateKey return NULL\n");

					ret = SSL_CTX_use_RSAPrivateKey(mosq->ssl_ctx, tls_rsa_key);

					RSA_free(tls_rsa_key);
					BIO_set_close(tls_key_bio, BIO_NOCLOSE);
					BIO_free(tls_key_bio);

					if(ret != 1)
					{
#ifdef WITH_BROKER
						log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to load client key file, check bridge_keyfile \"%s\".", mosq->tls_keyfile);
#else
						log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to load client key file \"%s\".", mosq->tls_keyfile);
#endif
#if !defined(OPENSSL_NO_ENGINE)
						ENGINE_FINISH(engine);
#endif
						COMPAT_CLOSE(mosq->sock);
						mosq->sock = INVALID_SOCKET;
						net__print_ssl_error(mosq);
						return MOSQ_ERR_TLS;
					}
				}
				ret = SSL_CTX_check_private_key(mosq->ssl_ctx); // 检查私钥和证书对不对的上
				if(ret != 1){
					log__printf(mosq, MOSQ_LOG_ERR, "Error: Client certificate/key are inconsistent.");
#if !defined(OPENSSL_NO_ENGINE)
					ENGINE_FINISH(engine);
#endif
					COMPAT_CLOSE(mosq->sock);
					mosq->sock = INVALID_SOCKET;
					net__print_ssl_error(mosq);
					return MOSQ_ERR_TLS;
				}
			}
#ifdef FINAL_WITH_TLS_PSK
		}else if(mosq->tls_psk){
			SSL_CTX_set_psk_client_callback(mosq->ssl_ctx, psk_client_callback);
#endif
		}
	}

	return MOSQ_ERR_SUCCESS;
}
```

从注释就可以看出来了，改的地方就两处：在注释“改为内存加载”那里。（参考：<<https://stackoverflow.com/questions/3810058/read-certificate-files-from-memory-instead-of-a-file-using-openssl>>）

还有其他一些小地方需要加的，头文件和 mosqquitto 实例结构体多加几个成员，还有对这几个成员的析构，具体如下：

头文件在`lib/mosquitto_internal.h`开头部分，

```c++
#ifndef MOSQUITTO_INTERNAL_H
#define MOSQUITTO_INTERNAL_H

#include "config.h"

#ifdef WIN32
#  include <winsock2.h>
#endif

#ifdef WITH_TLS
#  include <openssl/ssl.h>
#  include <openssl/bio.h> // 这里
#  include <openssl/pem.h> // 这里
#  include <openssl/rsa.h> // 这里
#else
#  include <time.h>
#endif
#include <stdlib.h>
```

