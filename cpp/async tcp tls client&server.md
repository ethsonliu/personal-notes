参考：https://codereview.stackexchange.com/questions/108600/complete-async-openssl-example

```c++
#include <stdio.h>
#include <string.h>
#include <openssl/ssl.h>
#include <openssl/crypto.h>
#include <openssl/err.h>
#include <fcntl.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/tcp.h>
#include <sys/select.h>
#include <netinet/in.h>

typedef enum {
  CONTINUE,
  BREAK,
  NEITHER
} ACTION;

ACTION ssl_connect(SSL* ssl, int* wants_tcp_write, int* connecting) {
  printf("calling SSL_connect\n");

  int result = SSL_connect(ssl);
  if (result == 0) {
    long error = ERR_get_error();
    const char* error_str = ERR_error_string(error, NULL);
    printf("could not SSL_connect: %s\n", error_str);
    return BREAK;
  } else if (result < 0) {
    int ssl_error = SSL_get_error(ssl, result);
    if (ssl_error == SSL_ERROR_WANT_WRITE) {
      printf("SSL_connect wants write\n");
      *wants_tcp_write = 1;
      return CONTINUE;
    }

    if (ssl_error == SSL_ERROR_WANT_READ) {
      printf("SSL_connect wants read\n");
      // wants_tcp_read is always 1;
      return CONTINUE;
    }

    long error = ERR_get_error();
    const char* error_string = ERR_error_string(error, NULL);
    printf("could not SSL_connect %s\n", error_string);
    return BREAK;
  } else {
    printf("connected\n");
    *connecting = 0;
    return CONTINUE;
  }

  return NEITHER;
}

ACTION ssl_read(SSL* ssl, int* wants_tcp_write, int* call_ssl_read_instead_of_write) {
  printf("calling SSL_read\n");

  *call_ssl_read_instead_of_write = 0;

  char buffer[1024];
  int num = SSL_read(ssl, buffer, sizeof(buffer));
  if (num == 0) {
    long error = ERR_get_error();
    const char* error_str = ERR_error_string(error, NULL);
    printf("could not SSL_read (returned 0): %s\n", error_str);
    return BREAK;
  } else if (num < 0) {
    int ssl_error = SSL_get_error(ssl, num);
    if (ssl_error == SSL_ERROR_WANT_WRITE) {
      printf("SSL_read wants write\n");
      *wants_tcp_write = 1;
      *call_ssl_read_instead_of_write = 1;
      return CONTINUE;
    }

    if (ssl_error == SSL_ERROR_WANT_READ) {
      printf("SSL_read wants read\n");
      // wants_tcp_read is always 1;
      return CONTINUE;
    }

    long error = ERR_get_error();
    const char* error_string = ERR_error_string(error, NULL);
    printf("could not SSL_read (returned -1) %s\n", error_string);
    return BREAK;
  } else {
    printf("read %d bytes\n", num);
  }

  return NEITHER;
}

ACTION ssl_write(SSL* ssl, int* wants_tcp_write, int* call_ssl_write_instead_of_read,
    int is_client, int should_start_a_new_write) {
  printf("calling SSL_write\n");

  static char buffer[1024];
  memset(buffer, 0, sizeof(buffer));
  static int to_write = 0;
  if (!*call_ssl_write_instead_of_read && !to_write && is_client && should_start_a_new_write) {
    to_write = 1024;
    printf("decided to write %d bytes\n", to_write);
  }

  if (*call_ssl_write_instead_of_read && (!to_write || !buffer)) {
    printf("ssl should not have requested a write from a read if no data was waiting to be written\n");
    return BREAK;
  }

  *call_ssl_write_instead_of_read = 0;

  if (!to_write) {
    return NEITHER;
  }

  int num = SSL_write(ssl, buffer, to_write);
  if (num == 0) {
    long error = ERR_get_error();
    const char* error_str = ERR_error_string(error, NULL);
    printf("could not SSL_write (returned 0): %s\n", error_str);
    return BREAK;
  } else if (num < 0) {
    int ssl_error = SSL_get_error(ssl, num);
    if (ssl_error == SSL_ERROR_WANT_WRITE) {
      printf("SSL_write wants write\n");
      *wants_tcp_write = 1;
      return CONTINUE;
    }

    if (ssl_error == SSL_ERROR_WANT_READ) {
      printf("SSL_write wants read\n");
      *call_ssl_write_instead_of_read = 1;
      // wants_tcp_read is always 1;
      return CONTINUE;
    }

    long error = ERR_get_error();
    const char* error_string = ERR_error_string(error, NULL);
    printf("could not SSL_write (returned -1): %s\n", error_string);
    return BREAK;
  } else {
    printf("wrote %d of %d bytes\n", num, to_write);
    if (to_write < num) {
      *wants_tcp_write = 1;
    } else {
      *wants_tcp_write = 0;
    }
    to_write -= num;
  }

  return NEITHER;
}

int main(int argc, char** argv) {
  if (argc != 2 || (strcmp(argv[1], "client") && strcmp(argv[1], "server"))) {
    printf("need parameter 'client' or 'server'\n");
    return 1;
  }

  int is_client = !strcmp(argv[1], "client");
  int port = 10000;

  SSL_library_init();
  SSL_load_error_strings();

  SSL_CTX* ssl_ctx = SSL_CTX_new(is_client ?
      SSLv23_client_method() :
      SSLv23_server_method());
  if (!ssl_ctx) {
    printf("could not SSL_CTX_new\n");
    return 1;
  }

  int sockfd = 0;
  if (is_client) {
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0) {
      printf("could not create socket\n");
      return 1;
    }
  } else {
    const char* certificate = "./cert.pem";
    if (SSL_CTX_use_certificate_file(ssl_ctx, certificate, SSL_FILETYPE_PEM) != 1) {
      printf("could not SSL_CTX_use_certificate_file\n");
      return 1;
    }

    if (SSL_CTX_use_PrivateKey_file(ssl_ctx, certificate, SSL_FILETYPE_PEM) != 1) {
      printf("could not SSL_CTX_use_PrivateKey_file\n");
      return 1;
    }

    int server = socket(AF_INET, SOCK_STREAM, 0);
    if (server < 0) {
      printf("could not create socket\n");
      return 1;
    }

    int on = 1;
    if (setsockopt(server, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(on))) {
      close(server);
      printf("could not setsockopt\n");
      return 1;
    }

    // Bind on any interface.
    struct sockaddr_in addr;
    bzero(&addr, sizeof(addr));
    addr.sin_family = AF_INET;
    addr.sin_port = htons(port);

    if (bind(server, (struct sockaddr*)(&addr),
      sizeof(addr))) {
      printf("could not bind\n");
      close(server);
      return 1;
    }

    if (listen(server, 1)) {
      printf("could not listen\n");
      close(server);
      return 1;
    }

    sockfd = accept(server, NULL, NULL);
    if (sockfd < 0) {
      printf("could not create accept\n");
      return 1;
    }
  }

  SSL* ssl = SSL_new(ssl_ctx);
  if (!ssl) {
    printf("could not SSL_new\n");
    return 1;
  }

  // Set the socket to be non blocking.
  int flags = fcntl(sockfd, F_GETFL, 0);
  if (fcntl(sockfd, F_SETFL, flags | O_NONBLOCK)) {
    printf("could not fcntl\n");
    close(sockfd);
    return 1;
  }

  int one = 1;
  if (setsockopt(sockfd, SOL_TCP, TCP_NODELAY, &one, sizeof(one))) {
    printf("could not setsockopt\n");
    close(sockfd);
    return 1;
  }

  if (is_client) {
    struct sockaddr_in addr;
    memset(&addr, 0, sizeof(addr));
    addr.sin_family = AF_INET;
    addr.sin_port = htons(port);
    addr.sin_addr.s_addr = htonl((in_addr_t)(0x7f000001));

    if (connect(sockfd, (struct sockaddr*)(&addr), sizeof(addr)) && errno != EINPROGRESS) {
      printf("could not connect\n");
      return 1;
    }
  }

  if (!SSL_set_fd(ssl, sockfd)) {
    close(sockfd);
    printf("could not SSL_set_fd\n");
    return 1;
  }

  int connecting = 1;
  if (is_client) {
    SSL_set_connect_state(ssl);
  } else {
    SSL_set_accept_state(ssl);
    connecting = 0;
  }

  fd_set read_fds, write_fds;
  int wants_tcp_read = 1, wants_tcp_write = is_client;
  int call_ssl_read_instead_of_write = 0;
  int call_ssl_write_instead_of_read = 0;

  for (;;) {
    printf("selecting\n");
    FD_ZERO(&read_fds);
    FD_ZERO(&write_fds);
    if (wants_tcp_read) {
      FD_SET(sockfd, &read_fds);
    }

    if (wants_tcp_write) {
      FD_SET(sockfd, &write_fds);
    }

    struct timeval timeout = { 1, 0 };

    if (select(sockfd + 1, &read_fds, &write_fds, NULL, &timeout)) {
      if (FD_ISSET(sockfd, &read_fds)) {
        printf("readable\n");

        if (connecting) {
          ACTION action = ssl_connect(ssl, &wants_tcp_write, &connecting);
          if (action == CONTINUE) {
            continue;
          } else if (action == BREAK) {
            break;
          }
        } else {
          ACTION action;
          if (call_ssl_write_instead_of_read) {
            action = ssl_write(ssl, &wants_tcp_write, &call_ssl_write_instead_of_read, is_client, 0);
          } else {
            action = ssl_read(ssl, &wants_tcp_write, &call_ssl_read_instead_of_write);
          }

          if (action == CONTINUE) {
            continue;
          } else if (action == BREAK) {
            break;
          }
        }
      }

      if (FD_ISSET(sockfd, &write_fds)) {
        printf("writable\n");

        if (connecting) {
          wants_tcp_write = 0;

          ACTION action = ssl_connect(ssl, &wants_tcp_write, &connecting);
          if (action == CONTINUE) {
            continue;
          } else if (action == BREAK) {
            break;
          }
        } else {
          ACTION action;
          if (call_ssl_read_instead_of_write) {
            action = ssl_read(ssl, &wants_tcp_write, &call_ssl_read_instead_of_write);
          } else {
            action = ssl_write(ssl, &wants_tcp_write, &call_ssl_write_instead_of_read, is_client, 0);
          }

          if (action == CONTINUE) {
            continue;
          } else if (action == BREAK) {
            break;
          }
        }
      }
    } else if (is_client & !connecting && !call_ssl_write_instead_of_read) {
      ACTION action = ssl_write(ssl, &wants_tcp_write, &call_ssl_write_instead_of_read, is_client, 1);
      if (action == CONTINUE) {
        continue;
      } else if (action == BREAK) {
        break;
      }
    }
  }

  SSL_CTX_free(ssl_ctx);

  return 0;
}
```
