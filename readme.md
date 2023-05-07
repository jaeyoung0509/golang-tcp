## tcp 


### net.listen
```go
func Listen(network, address string) (Listener ,error)
```
- Network types: `tcp`, `tcp4`, `tcp6` `unix`, `unixpacket`
  - tcp4 means ipv4
- `address` : local network address and there are several patterns for tcp networks

![inner implementation](/images/2023-05-07-11-04-18.png)
- [ref](https://dev.to/hgsgtk/how-go-handles-network-and-system-calls-when-tcp-server-1nbd)
- net.listen invokes ListenConfig.Listen
- then it creates DefaultResolver which type is net.Resolver
who actually resolves the network Ip address
> Resolver
> - 주로 호스트 이름을 Ip 주소로 변환하는 기능을 수행하는 구성요소 
> - 이러한 변환 과정을 `Dns resolution` `name resolution` 
> -  `net.Resolver` 구초체는 dns 쿼리를 수행하는데 사용되며 다양한 dns 조회 메서드를 제공함 
> -  LookupHost, LookupIP, LookupCNAME, LookupMX, LookupNS, LookupSRV, LookupTXT 등의 메서드가 있음
``` go 
type Resolver struct {
    PreferGo bool
    StrictErrors bool
    Dial func(ctx context.Context , network, address string) (Conn, error)
}

```

### dynamically choose a port
![](images/2023-05-07-11-13-37.png)
- net.Listen invokes net.ListenConfig.Listen
- net.Resolver.resolveAddrList resolves the address as `0` if the port in the address
parameter is empty or `0`
- net.ListenConfig.Listen creates a net.SyncListener contains a Listen's parameters and configuration
When the network protocol is tcp, net.ListenConfig.Listen invokes net.sysListener.listenTCP
```go 
func (l *ListenConfig) Listen(ctx context.Context, network, addr string) (net.Listener, error) {
    addrs, err := DefaultResolver.resolveAddrList(ctx, "listen", address, nil)
    sl := &sysListener{
        ListenConfig: *lc,
        network: network,
        address:address,
    }
    var l Listener 
    la := addrs.first(isIPv4)
    switch la := la.(type) {
        case *TCPAddr:
            l, err := sl.listenTCP(ctx, la)
        case *UnixAddr:
            l, err := sl.ListenUnix(ctx, la)
    }
}
```


### tcp io 
- read: receive
- write: send
