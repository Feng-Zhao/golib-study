# bufio.Writer

[toc]

## Writer 结构

```go
// 1. 当 write 动作出现 err 时,后续的 write, flush 都会报错
// 2. 所有数据都写入完成后, 需要 flush 来保证最后一部分数据被刷到 io.Writer
type Writer struct {
 err error
 buf []byte
 n   int        // 已缓存的 byte 数
 wr  io.Writer
}
```

## 实例化

```go
func NewWriter(w io.Writer) *Writer
func NewWriterSize(w io.Writer, size int) *Writer
```

## Writer 方法

### Available 返回 buf 中有多少 byte 还没被使用(缓存空余量)

```go
// buf 中有多少 byte 还没被使用
func (b *Writer) Available() int { return len(b.buf) - b.n }
```

### Buffered 返回已经缓存的字节数(包括已使用的和未使用的)

```go
func (b *Writer) Buffered() int { return b.n }
```

### Size 返回 buf 大小

```go
func (b *Writer) Size() int {return len(b.buf)}
```

### Flush 刷盘 重要,完成 write 后,需要 flush 将数据从缓存区持久化到磁盘

```go
func (b *Writer) Flush() error {
    if b.err != nil {
        return b.err
    }
    if b.n == 0 {
        return nil
    }
    n, err := b.wr.Write(b.buf[0:b.n])
    if n < b.n && err == nil {
        err = io.ErrShortWrite
    }
    if err != nil {
        if n > 0 && n < b.n {
            copy(b.buf[0:b.n-n], b.buf[n:b.n])
        }
        b.n -= n
        b.err = err
        return err
    }
    b.n = 0
    return nil
}
```

### Write 系列方法

#### Write 写入 []byte

将 content 写入缓存区

```go
func (b *Writer) Write(content []byte) (nn int, err error)
```

#### WriteString 写入 string

```go
func (b *Writer) WriteString(s string) (int, error)
```

#### WriteByte 写入单个 byte

```go
func (b *Writer) WriteByte(c byte) error
```

#### WriteRune 写入单个 rune

```go
func (b *Writer) WriteRune(r rune) (size int, err error)
```

### Reset

```go
func (b *Writer) Reset(w io.Writer)
```

### ReadFrom

```go
func (b *Writer) ReadFrom(r io.Reader) (n int64, err error)
```
