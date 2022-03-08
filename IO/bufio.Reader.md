# bufio.Reader

[toc]

## Reader 结构

```go
type Reader struct {
    buf          []byte        // 缓存
    rd           io.Reader    // 底层的io.Reader
    // r:从buf中读走的字节（偏移）；w:buf中填充内容的偏移；
    // w - r 是buf中可被读的长度（缓存数据的大小），也是Buffered()方法的返回值
    // r 即为游标 cursor, w 为 cap 
    r, w         int
    err          error      // 读过程中遇到的错误
    lastByte     int        // 最后一次读到的字节（ReadByte/UnreadByte)
    lastRuneSize int        // 最后一次读到的Rune的大小 (ReadRune/UnreadRune)
}
// 常量
MaxScanTokenSize = 64 * 1024 // 最大 token size = 8KB
```

## NewReader 实例化

```go
// 实例化
func NewReader(rd io.Reader) *Reader {
    // 默认缓存大小：defaultBufSize=4096
    return NewReaderSize(rd, defaultBufSize)
}
// 实际上是调用 NewReaderSize
func NewReaderSize(rd io.Reader, size int) *Reader {
    // 已经是bufio.Reader类型，且缓存大小不小于 size，则直接返回
    b, ok := rd.(*Reader)
    if ok && len(b.buf) >= size {
        return b
    }
    // 缓存大小不会小于 minReadBufferSize （16字节）
    if size < minReadBufferSize {
        size = minReadBufferSize
    }
    // 构造一个bufio.Reader实例
    return &Reader{
        buf:          make([]byte, size),
        rd:           rd,
        lastByte:     -1,
        lastRuneSize: -1,
    }
}
```

## Reader 方法

### Buffered 查看可读缓存大小 w - r

```go
func (b *Reader) Buffered() int
```

### Size 返回 buf 大小 len(buf)

```go
func (b *Reader) Size() int
```

### Discard 跳过 n 个字节

```go
func (b *Reader) Discard(n int) (discarded int, err error)
```

### Peek 查看栈顶

```go
func (b *Reader) Peek(n int) ([]byte, error)
```

### Read 读取

#### Read 读取数据

将 Reader 缓存的数据读取到 p[]byte 中, 返回 n 表示读取了多少字节.
这个数量根据 Reader 中包装的 io.Reader 有关
若想装满 p,则需使用 io.ReadFull(b, p) 方法
在 EOF 时,会返回 0, io.EOF

```go
func (b *Reader) Read(p []byte) (n int, err error)
```

#### ReadString 读取 string // 常用

```go
func (b *Reader) ReadString(delim byte) (string, error)
```

#### ReadByte 读取字节 // 常用

```go
// 单个字节
func (b *Reader) ReadByte() (byte, error)
// 读取起始位置-第一次遇到 delim 的所有字节,返回 slice
func (b *Reader) ReadBytes(delim byte) ([]byte, error)
```

#### ReadRune 读取字符

```go
func (b *Reader) ReadRune() (r rune, size int, err error)
```

#### ReadLine 读取行

这是个基础方法,尽量不要使用这个方法,应使用

- `ReadBytes('\n') // 对数据型数据`
- `ReadString('\n') // 对字符型数据`
- `Scanner`

ReadLine 返回值:
line: 读取的行
isPrefix: true -> 行数据过大,未读完 | false -> 行的最后一部分

此方法的返回不包含 行的终结符 `\n\r` 和 `\n` 但是读取完成后会跳过行终结符

```go
func (b *Reader) ReadLine() (line []byte, isPrefix bool, err error)
```

#### ReadSlice 方法

这是个基础方法,尽量也不要用这个方法,
和上边类似,但是这个方法会返回行终结符
err 只在 行未使用 delim 结束的情况下有值,通常为 io.EOF,另外的情况还有 ErrBufferFull

```go
// delim 为界定符, 返回从 buf 可读到第一次遇到 delim, 且包含 delim 的 []byte
func (b *Reader) ReadSlice(delim byte) (line []byte, err error)
```

### Reset 重置

使用 r io.Reader 重新包装

```go
func (b *Reader) Reset(r io.Reader)
```

### Unread 系列

```go
func (b *Reader) UnreadByte() error // 上一个操作不是 Read 操作时,报错
func (b *Reader) UnreadByte() error // 上一个操作不是 ReadRune 操作时,报错
```

### WriteTo

````go
func (b *Reader) WriteTo(w io.Writer) (n int64, err error)
```
