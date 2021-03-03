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
// 服务端接收
package main

import (
	"net/http"
	"net"
	"fmt"
	"io"
)



func handle(w http.ResponseWriter, req *http.Request) {
	buf := make([]byte, 256)
	var n int
	for {
		n, err := req.Body.Read(buf)
		if err == io.EOF {
			break
		}
		fmt.Printf(string(buf[:n]))
	}
	fmt.Printf(string(buf[:n]))
	fmt.Printf("TRANSMISSION COMPLETE")
}

func main() {
	/* Net listener */
	n := "tcp"
	addr := "127.0.0.1:9094"
	l, err := net.Listen(n, addr)
	if err != nil {
		panic("AAAAH")
	}

	/* HTTP server */
	server := http.Server{
		Handler: http.HandlerFunc(handle),
	}
	server.Serve(l)
}
```

接收的代码还可以参考 <https://stackoverflow.com/questions/59170937/golang-parsing-chunked-http-responses-from-socket-read-chunk-by-chunk>

## 参考

- <https://stackoverflow.com/questions/26769626/send-a-chunked-http-response-from-a-go-server>
- <https://gist.github.com/yinchunxiang/ef170ce11927a2f057a32870e2dd4cd2>
- <https://stackoverflow.com/questions/19267336/golang-http-handle-big-file-upload>
- <https://gist.github.com/ZenGround0/af448f56882c16aaf10f39db86b4991e>
