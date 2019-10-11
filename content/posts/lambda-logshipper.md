+++
title = "Lambda Logshipper"
date = "2019-07-11"
aliases = ["Post","post","blog-page"]
tags = ["Golang","Lambda", "AWS", "Cloudwatch", "cloud admin"]
disqusShortname = "http-applegreengrape-com"
+++

So how can you easily move your Cloudwatch logstream to another platform or log collector endpoints? The easiest way is to ship the Cloudwatch logstream through a socket client. This is an example of a small Golang lambda function to ship aws cloudwatch log stream to a tcp endpoint.

you will need a socket client:
```go
func SocketClient(m []byte) {
	conn, err := net.Dial("tcp", "your_tcp_endpoint:your_port")

	defer conn.Close()

	if err != nil {
		fmt.Println(err)
	}

	fmt.Println("sending logs to syslog-ng:", string(m))
	conn.Write(m)

	buff := make([]byte, 1024)
	n, _ := conn.Read(buff)
	log.Printf("Receive: %s", buff[:n])

}
```
N.B. SocketClient is easy for shipping the data. But maybe not the most secured way. Adding a tls cert might be a better idea 😃.

You will also need a go client for Cloudwatch to recieve and parse the incoming messages.
```go
import (
    ...
	"github.com/aws/aws-lambda-go/events"
	"github.com/aws/aws-lambda-go/lambda"
)

func Handler(request events.CloudwatchLogsEvent) error {
	cloudwatchLogsData, err := request.AWSLogs.Parse()
	if err != nil {
		fmt.Println(err)
		return nil
	}

	fmt.Println("cloudwatchLogsData.LogEvents:", cloudwatchLogsData.LogEvents)

	// Remove the prefix to get to the name of the Lambda.
	logGroup := strings.Replace(cloudwatchLogsData.LogGroup, "/aws/lambda/", "", 1)

	// What you want to capture is up to you. BUT, for example:
	type LogEntry struct {
		LogGroup  string `json:"log_group"`
		Timestamp int64  `json:"timestamp"`
		Message   string `json:"message"`
	}

	// Stuff the incoming log lines into the datastructure to serialize to Log Entries.
	for _, event := range cloudwatchLogsData.LogEvents {
		logEntry := LogEntry{LogGroup: logGroup, Timestamp: event.Timestamp, Message: event.Message}
	//	logEntry := LogEntry{Timestamp: event.Timestamp, Message: event.Message}
		j, err := json.Marshal(logEntry)
		if err != nil {
			fmt.Println(err)
			return nil
		}
		
		// Send to syslog-ng over tcp, basically.
		SocketClient(j)
		fmt.Println(j)
	}

	return nil
}
```

You can find the [source code here](https://github.com/applegreengrape/go-aws-lambda-logshipper)