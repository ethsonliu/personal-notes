首先找到设置证书和私钥的位置，下面这个接口，位置在：`libs/options.c`，修改如下，

```c++
int mosquitto_tls_set(struct mosquitto *mosq, const char *cafile, const char *capath, const char *certfile, const char *keyfile, int (*pw_callback)(char *buf, int size, int rwflag, void *userdata))
{
#ifdef WITH_TLS
	FILE *fptr;

	if(!mosq || (!cafile && !capath) || (certfile && !keyfile) || (!certfile && keyfile)) return MOSQ_ERR_INVAL;

	mosquitto__free(mosq->tls_cafile);
	mosq->tls_cafile = NULL;
	if(cafile){
		fptr = mosquitto__fopen(cafile, "rt", false); // 根证书名
		if(fptr){
			fclose(fptr);
		}else{
			return MOSQ_ERR_INVAL;
		}
		mosq->tls_cafile = mosquitto__strdup(cafile);

		if(!mosq->tls_cafile){
			return MOSQ_ERR_NOMEM;
		}
	}

	mosquitto__free(mosq->tls_capath);
	mosq->tls_capath = NULL;
	if(capath){
		mosq->tls_capath = mosquitto__strdup(capath); // 根证书所在路径
		if(!mosq->tls_capath){
			return MOSQ_ERR_NOMEM;
		}
	}

	mosquitto__free(mosq->tls_certfile);
	mosq->tls_certfile = NULL;
	if(certfile)
	{
		mosq->tls_certfile = mosquitto__strdup(certfile); // 客户端证书
		if(!mosq->tls_certfile)
			return MOSQ_ERR_NOMEM;
	}
	else
	{
		mosquitto__free(mosq->tls_cafile);
		mosq->tls_cafile = NULL;

		mosquitto__free(mosq->tls_capath);
		mosq->tls_capath = NULL;
		return MOSQ_ERR_INVAL;
	}

	mosquitto__free(mosq->tls_keyfile);
	mosq->tls_keyfile = NULL;
	if(keyfile)
	{
		mosq->tls_keyfile = mosquitto__strdup(keyfile); // 客户端私钥
		if(!mosq->tls_keyfile)
			return MOSQ_ERR_NOMEM;
	}
	else
	{
		mosquitto__free(mosq->tls_cafile);
		mosq->tls_cafile = NULL;

		mosquitto__free(mosq->tls_capath);
		mosq->tls_capath = NULL;

		mosquitto__free(mosq->tls_certfile);
		mosq->tls_certfile = NULL;
		return MOSQ_ERR_INVAL;
	}

	mosq->tls_pw_callback = pw_callback;


	return MOSQ_ERR_SUCCESS;
#else
	return MOSQ_ERR_NOT_SUPPORTED;

#endif
}
```

这个接口只是简单的做些文件判断存不存在，如果存在就把文件路径赋给 mosq 实体。

接下来具体在哪里处理这些文件的呢？

`lib/net_mosq.c` 中的 `static int net__init_ssl_ctx(struct mosquitto *mosq)` 中实现了主要 SSL 处理，

改造后的代码为：

```c
if(mosq->tls_certfile){
	// 改为内存加载
	BIO  *tls_cert_bio = BIO_new_mem_buf((void*)mosq->tls_certfile, -1);
	if (tls_cert_bio == NULL)
		log__printf(mosq, MOSQ_LOG_ERR, "Error: [net__init_ssl_ctx] BIO_new_mem_buf(tls_certfile) return NULL\n");

	X509 *tls_cert_x509 = PEM_read_bio_X509(tls_cert_bio, NULL, 0, NULL);
	if (tls_cert_x509 == NULL)
		log__printf(mosq, MOSQ_LOG_ERR, "[net__init_ssl_ctx] PEM_read_bio_X509 return NULL\n");

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
		net__print_ssl_error(mosq);
		return MOSQ_ERR_TLS;
	}
}

if(mosq->tls_keyfile){
	if(mosq->tls_keyform == mosq_k_engine){
#if !defined(OPENSSL_NO_ENGINE)
        UI_METHOD *ui_method = net__get_ui_method();
        if(mosq->tls_engine_kpass_sha1){
        	if(!ENGINE_ctrl_cmd(engine, ENGINE_SECRET_MODE, ENGINE_SECRET_MODE_SHA, NULL, NULL, 0)){
        		log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to set engine secret mode sha1");
        		ENGINE_FINISH(engine);
        		net__print_ssl_error(mosq);
        		return MOSQ_ERR_TLS;
        	}
        	if(!ENGINE_ctrl_cmd(engine, ENGINE_PIN, 0, mosq->tls_engine_kpass_sha1, NULL, 0)){
        		log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to set engine pin");
        		ENGINE_FINISH(engine);
        		net__print_ssl_error(mosq);
        		return MOSQ_ERR_TLS;
        	}
        	ui_method = NULL;
        }
        pkey = ENGINE_load_private_key(engine, mosq->tls_keyfile, ui_method, NULL);
        if(!pkey){
        	log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to load engine private key file \"%s\".", mosq->tls_keyfile);
        	ENGINE_FINISH(engine);
        	net__print_ssl_error(mosq);
        	return MOSQ_ERR_TLS;
        }
        if(SSL_CTX_use_PrivateKey(mosq->ssl_ctx, pkey) <= 0){
        	log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to use engine private key file \"%s\".", mosq->tls_keyfile);
        	ENGINE_FINISH(engine);
        	net__print_ssl_error(mosq);
        	return MOSQ_ERR_TLS;
        }
#endif
	}else{
		// 改为内存加载
		BIO  *tls_key_bio = BIO_new_mem_buf((void*)mosq->tls_keyfile, -1);
		if (tls_key_bio == NULL)
			log__printf(mosq, MOSQ_LOG_ERR, "[net__init_ssl_ctx] BIO_new_mem_buf(tls_keyfile) return NULL\n");

		RSA  *tls_rsa_key = PEM_read_bio_RSAPrivateKey(tls_key_bio, NULL, 0, NULL);
		if (tls_rsa_key == NULL)
			log__printf(mosq, MOSQ_LOG_ERR, "[net__init_ssl_ctx] PEM_read_bio_RSAPrivateKey return NULL\n");

		ret = SSL_CTX_use_RSAPrivateKey(mosq->ssl_ctx, tls_rsa_key);

		RSA_free(tls_rsa_key);
		BIO_set_close(tls_key_bio, BIO_NOCLOSE);
		BIO_free(tls_key_bio);

		if(ret != 1){
#ifdef WITH_BROKER
			log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to load client key file, check bridge_keyfile \"%s\".", mosq->tls_keyfile);
#else
			log__printf(mosq, MOSQ_LOG_ERR, "Error: Unable to load client key file \"%s\".", mosq->tls_keyfile);
#endif
#if !defined(OPENSSL_NO_ENGINE)
			ENGINE_FINISH(engine);
#endif
			net__print_ssl_error(mosq);
			return MOSQ_ERR_TLS;
		}
	}
	ret = SSL_CTX_check_private_key(mosq->ssl_ctx);
    if(ret != 1){
		log__printf(mosq, MOSQ_LOG_ERR, "Error: Client certificate/key are inconsistent.");
#if !defined(OPENSSL_NO_ENGINE)
		ENGINE_FINISH(engine);
#endif
		net__print_ssl_error(mosq);
		return MOSQ_ERR_TLS;
	}
}
```

从注释就可以看出来了，改的地方就两处：在注释 “改为内存加载” 那里。参考：<https://stackoverflow.com/questions/3810058/read-certificate-files-from-memory-instead-of-a-file-using-openssl>

还有其他一些小地方需要加的，添加一些需要 include 的头文件，在`lib/mosquitto_internal.h`开头部分，

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

还有一处，这是在我做 windows 端客户端开发的时候发现的。某些情况下，mosq 无法自动重连 server，跟踪发现是以下这个位置的问题，`lib/loop.c` 中的 `mosquitto_loop_forever` 修改如下，

```c
#ifndef WIN32
		if(errno == EPROTO){
			return rc;
		}
#endif
```

在此次改造中还发现，keepalive 的值不能小于 5，否则会报错，参见 <https://github.com/eclipse/mosquitto/commit/8e7e4a9d9a682c710268e050d3c3db1e9e372501>。
