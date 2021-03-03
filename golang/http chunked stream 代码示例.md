```golang
// server 端
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
// client 端
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

## 参考

- <https://stackoverflow.com/questions/26769626/send-a-chunked-http-response-from-a-go-server>
- <https://gist.github.com/yinchunxiang/ef170ce11927a2f057a32870e2dd4cd2>
