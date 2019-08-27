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
    // 查看文件是否能够打开，若能打开
    mosq->tls_cafile   = mosquitto__strdup(cafile);
    mosq->tls_capath   = mosquitto__strdup(capath);
    mosq->tls_certfile = mosquitto__strdup(certfile);
    mosq->tls_keyfile  = mosquitto__strdup(keyfile);
}
```

在`lib/net_mosq.c`中的`static int net__init_ssl_ctx(struct mosquitto *mosq)`中实现了主要 SSL 处理，

