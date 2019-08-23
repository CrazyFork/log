# Notes

-> 目标
* 如何实现一个logger, 

就tj第一版(initial commit)的实现来看, 比我预想中的要简单. 可能唯一需要注意的就是 io 在做
output 的时候注意加锁.

-> key component

* Fields
* Entry
* Logger
* Handler


* Entry, a log item entry. 
  ```go
  try represents a single log entry.
  type Entry struct {
    Logger    *Logger   `json:"-"`
    Fields    Fields    `json:"fields"`
    Level     Level     `json:"level"`
    Timestamp time.Time `json:"timestamp"`
    Message   string    `json:"message"`
  }
  ```

* Fields, a Entry has many fields, `type Fields Map<String, any>` 

* Level, log level

* Handler, `type Handler = F(Entry)=>error`

-> 

* Trace, field 有complete字段, type boolean, [ true, false ]
* pkg.go, 里边声明了一个全局的logger,  

-> dirs

```plain
├── default.go                          // 默认std logger的handler输出函数, 会将 Fields name 排序输出
├── doc.go                              // 文档入口
├── entry.go                            // Entry 定义
├── entry_test.go
├── handlers                            // 用于控制日志如何输出的
│   ├── papertrail
│   │   └── papertrail.go
│   └── text
│       ├── text.go
│       └── text_test.go
├── interface.go                        // interface 总览
├── levels.go                           // log Level 定义
├── levels_test.go
├── logger.go                           // Logger 定义
├── logger_test.go
├── pkg.go                              // pkg 定义
├── pkg_test.go
├── stack.go                            // stack type 定义

```

-> Notes 



-> 

// go interface 是松散型的, 意味着你实现如下方法就行
type Handler interface {
	HandleLog(*Entry) error
}


// assert interface compliance.
var _ Interface = (*Logger)(nil)


-> 
func (e *Entry) WithFields(fields Fielder) *Entry {
	f := Fields{}

	for k, v := range e.Fields {
		f[k] = v
	}

	for k, v := range fields.Fields() {
		f[k] = v
	}

// create new Entry
	return &Entry{Logger: e.Logger, Fields: f}
}

-> Sprintf ?

func (e *Entry) WithError(err error) *Entry {
	ctx := e.WithField("error", err.Error())

  // runtime type assertion
	if s, ok := err.(stackTracer); ok {
		frame := s.StackTrace()[0]

    // 这是干啥的??
		name := fmt.Sprintf("%n", frame)
		file := fmt.Sprintf("%+s", frame)
		line := fmt.Sprintf("%d", frame)

		parts := strings.Split(file, "\n\t")
		if len(parts) > 1 {
			file = parts[1]
		}

		ctx = ctx.WithField("source", fmt.Sprintf("%s: %s:%s", name, file, line))
	}

	if f, ok := err.(Fielder); ok {
		ctx = ctx.WithFields(f.Fields())
	}

	return ctx
}




![Structured logging for golang](assets/title.png)

Package log implements a simple structured logging API inspired by Logrus, designed with centralization in mind. Read more on [Medium](https://medium.com/@tjholowaychuk/apex-log-e8d9627f4a9a#.rav8yhkud).

## Handlers

- __cli__ – human-friendly CLI output
- __discard__ – discards all logs
- __es__ – Elasticsearch handler
- __graylog__ – Graylog handler
- __json__ – JSON output handler
- __kinesis__ – AWS Kinesis handler
- __level__ – level filter handler
- __logfmt__ – logfmt plain-text formatter
- __memory__ – in-memory handler for tests
- __multi__ – fan-out to multiple handlers
- __papertrail__ – Papertrail handler
- __text__ – human-friendly colored output
- __delta__ – outputs the delta between log calls and spinner

---

[![Build Status](https://semaphoreci.com/api/v1/projects/d8a8b1c0-45b0-4b89-b066-99d788d0b94c/642077/badge.svg)](https://semaphoreci.com/tj/log)
[![GoDoc](https://godoc.org/github.com/apex/log?status.svg)](https://godoc.org/github.com/apex/log)
![](https://img.shields.io/badge/license-MIT-blue.svg)
![](https://img.shields.io/badge/status-stable-green.svg)

<a href="https://apex.sh"><img src="http://tjholowaychuk.com:6000/svg/sponsor"></a>
