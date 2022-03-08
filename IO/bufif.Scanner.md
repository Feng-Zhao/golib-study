# bufio.Scanner

[toc]

## Scanner 结构

```go

// Scanner 提供对数据的 by line, by byte, by rune, by word 方式的读取,也支持自定义的 split func. 默认的 SplitFunc 是按行读取
// scan 操作出现 err 的情况有: EOF, token 过大
// scan 出错时是不可恢复的(即,没有 unread 系列的方法把游标往回调整),需要此部分功能的则需使用 bufio.Reader
type Scanner struct {
    r            io.Reader // The reader provided by the client.
    split        SplitFunc // The function to split the tokens.
    maxTokenSize int       // Maximum size of a token; modified by tests.
    token        []byte    // Last token returned by split.
    buf          []byte    // Buffer used as argument to split.
    start        int       // First non-processed byte in buf.
    end          int       // End of data in buf.
    err          error     // Sticky error.
    empties      int       // Count of successive empty tokens.
    scanCalled   bool      // Scan has been called; buffer is in use.
    done         bool      // Scan has finished.
}
```

## Scanner 实例化

```go
// 实例化
func NewScanner(r io.Reader) *Scanner
// 设置 buf, 必须在 scan 开始前设置 buf, 否则会 panic
func (s *Scanner) Buffer(buf []byte, max int)
// 设置 SplitFunc, 必须在 scan 开始前设置 buf, 否则会 panic
func (s *Scanner) Split(split SplitFunc)
```

## Scan 系列

### Scan

调用 `Scan()` 开始扫描, 返回 `true` 时,使用 `Bytes()` 或 `Text()` 获取 token
`Scan()` 返回 `false` 时,scan 结束, `Err()`会返回 err,但是 `io.EOF` 的情况除外(正常扫描到文件尾)

```go
func (s *Scanner) Scan() bool
```

### Bytes -> []byte

```go
func (s *Scanner) Bytes() []byte
```

### Text -> string

```go
func (s *Scanner) Text() string
```

## Err

```go
func (s *Scanner) Err() error
```

## SplitFunc

```go
type SplitFunc func(data []byte, atEOF bool) (advance int, token []byte, err error)
```
