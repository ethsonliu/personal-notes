```golang
// server 端发送
func main() {
  http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    flusher, ok := w.(http.Flusher)
    if !ok {
      panic("expected http.ResponseWriter to be an http.Flusher")
    }
    
    w.Header().Set("X-Content-Type-Options", "nosniff")
    for i := 1; i <= 10; i++ {
      fmt.Fprintf(w, "Chunk #%d\n", i)
      flusher.Flush() // Trigger "chunked" encoding and send a chunk...
      time.Sleep(500 * time.Millisecond)
    }
  })

  log.Print("Listening on localhost:8080")
  log.Fatal(http.ListenAndServe(":8080", nil))
}
```

```golang
// client 端发送
package main

import (
	"fmt"
	"io"
	"io/ioutil"
	"net/url"

	"github.com/benburkert/http"
	//"net/http"
	//"encoding/binary"
	"os"
)

const (
	chunk1 = "First Chunk"
	chunk2 = "Second Chunk"
)

func main() {
	rd, wr := io.Pipe()

	//u, _ := url.Parse("http://httpbin.org/post?show_env=1")
	//u, _ := url.Parse("http://requestb.in/zox5gczo")
	u, _ := url.Parse("https://httpbin.org/post")

	req := &http.Request{
		Method:           "POST",
		ProtoMajor:       1,
		ProtoMinor:       1,
		URL:              u,
		TransferEncoding: []string{"chunked"},
		Body:             rd,
		Header: 			make(map[string][]string),
	}
	req.Header.Set("Content-Type", "audio/pcm;bit=16;rate=8000")

	client := http.DefaultClient

	go func() {
		buf := make([]byte, 300)
		f, _ := os.Open("./tq.pcm")
		for {
			n, _ := f.Read(buf)
			if 0 == n {
				break
			}
			wr.Write(buf)
		}
		wr.Close()

	}()

	resp, err := client.Do(req)
	if nil != err {
		fmt.Println("error =>", err.Error())
		return
	}
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	if nil != err {
		fmt.Println("error =>", err.Error())
	} else {
		fmt.Println(string(body))
	}

}
```

```golang
// 客户端发送
package main

import (
	"bufio"
	"fmt"
	"io/ioutil"
	"net"
	"net/http"
	"os"
	"time"
)

// Precomputed http request
var data []string = []string{
	"PUT / HTTP/1.1\r\nHost: doesntmatter\r\nConnection: close\r\nTransfer-Encoding: chunked\r\n\r\n1\r\na\r\n",
	"1\r\nb\r\n0\r\n\r\n",
}

func main() {
	if len(os.Args) != 2 {
		fmt.Printf("Usage: %s host:port\n\tExample: %s 127.0.0.1:80\n", os.Args[0], os.Args[0])
		os.Exit(1)
	}
	for {
		// Open a tcp socket to the target
		soc, err := net.DialTimeout("tcp", os.Args[1], 50*time.Millisecond)
		if err != nil {
			if nerr, ok := err.(net.Error); ok {
				if nerr.Timeout() {
					fmt.Printf("Connection timed out to %s\n",os.Args[1])
					time.Sleep(time.Second)
				}
				continue
			}
			panic(err)
		}
		// Send frames one-by-one
		for _, frame := range data {
			if n, err := soc.Write([]byte(frame)); err != nil {
				panic(err)
			} else if n != len(frame) {
				panic("Short write")
			}
		}
		// Drain the socket in the background and compare the result.
		go func() {
			defer soc.Close()
			buf := bufio.NewReader(soc)
			response, err := http.ReadResponse(buf, nil)
			if err != nil {
				// Ignore errors
				return
			}
			content, err := ioutil.ReadAll(response.Body)
			if err != nil {
				return
			}
			fmt.Printf("Received code:%d body:%q\n", response.StatusCode, content)
		}()
		// Throtteling
		time.Sleep(100 * time.Millisecond)
	}
}
```

```golang
// 服务端接收
package main

import (
	"bytes"
	"fmt"
	"io"
	"net/http"
	"os"
	"strconv"
)

// A simple echo handler that replies the request body.
func echoHandler(w http.ResponseWriter, r *http.Request) {
	buf := &bytes.Buffer{}
	n, err := io.Copy(buf, r.Body)
	if err != nil {
		panic(err.Error())
	}
	fmt.Printf("Received body:%q\n", buf.Bytes())
	w.Header().Add("Content-Length", strconv.FormatInt(n, 10))
	w.WriteHeader(200)
	io.Copy(w, buf)
}

func main() {
	if len(os.Args) != 2 {
		fmt.Printf("Usage: %s ip:port\n\tExample: %s 0.0.0.0:80\n", os.Args[0], os.Args[0])
		os.Exit(1)
	}
	http.HandleFunc("/", echoHandler)
	err := http.ListenAndServe(os.Args[1], nil)
	if err != nil {
		panic(err)
	}
}
```

## 参考

- <https://stackoverflow.com/questions/26769626/send-a-chunked-http-response-from-a-go-server>
- <https://gist.github.com/yinchunxiang/ef170ce11927a2f057a32870e2dd4cd2>
- <https://stackoverflow.com/questions/19267336/golang-http-handle-big-file-upload>
- <https://gist.github.com/hannesg/541935ab2e4acc55ed8b85d65defe003>
- <https://stackoverflow.com/questions/59170937/golang-parsing-chunked-http-responses-from-socket-read-chunk-by-chunk>
- <https://gist.github.com/ZenGround0/af448f56882c16aaf10f39db86b4991e>
