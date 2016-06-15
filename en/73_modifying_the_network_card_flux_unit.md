# 7.3 Modifying the network card flux unit

Its unit is Bytes at present, and if you want to change it into Bits (such as: Mbps), you may modify the agent code: 

The code of original network card flux data collection  github.com/toolkits/nux/ifstat.go, is as follows:
```
func NetIfs(onlyPrefix []string) ([]*NetIf, error) {
    ...

    {

        ...

        netIf.InBytes, _ = strconv.ParseInt(fields[0], 10, 64)
        netIf.InPackages, _ = strconv.ParseInt(fields[1], 10, 64)
        netIf.InErrors, _ = strconv.ParseInt(fields[2], 10, 64)
        netIf.InDropped, _ = strconv.ParseInt(fields[3], 10, 64)
        netIf.InFifoErrs, _ = strconv.ParseInt(fields[4], 10, 64)
        netIf.InFrameErrs, _ = strconv.ParseInt(fields[5], 10, 64)
        netIf.InCompressed, _ = strconv.ParseInt(fields[6], 10, 64)
        netIf.InMulticast, _ = strconv.ParseInt(fields[7], 10, 64)

        netIf.OutBytes, _ = strconv.ParseInt(fields[8], 10, 64)
        netIf.OutPackages, _ = strconv.ParseInt(fields[9], 10, 64)
        netIf.OutErrors, _ = strconv.ParseInt(fields[10], 10, 64)
        netIf.OutDropped, _ = strconv.ParseInt(fields[11], 10, 64)
        netIf.OutFifoErrs, _ = strconv.ParseInt(fields[12], 10, 64)
        netIf.OutCollisions, _ = strconv.ParseInt(fields[13], 10, 64)
        netIf.OutCarrierErrs, _ = strconv.ParseInt(fields[14], 10, 64)
        netIf.OutCompressed, _ = strconv.ParseInt(fields[15], 10, 64)

        netIf.TotalBytes = netIf.InBytes + netIf.OutBytes
        netIf.TotalPackages = netIf.InPackages + netIf.OutPackages
        netIf.TotalErrors = netIf.InErrors + netIf.OutErrors
        netIf.TotalDropped = netIf.InDropped + netIf.OutDropped

        ret = append(ret, &netIf)
    }

    return ret, nil
}
```
The corresponding code of agent github.com/open-falcon/agent/funcs/ifstats.go is as follows:
```
func CoreNetMetrics(ifacePrefix []string) []*model.MetricValue {

    netIfs, err := nux.NetIfs(ifacePrefix)
    if err != nil {
        log.Println(err)
        return []*model.MetricValue{}
    }

    cnt := len(netIfs)
    ret := make([]*model.MetricValue, cnt*20)

    for idx, netIf := range netIfs {
        iface := "iface=" + netIf.Iface
        ret[idx*20+0] = CounterValue("net.if.in.bytes", netIf.InBytes, iface)
        ret[idx*20+1] = CounterValue("net.if.in.packets", netIf.InPackages, iface)
        ret[idx*20+2] = CounterValue("net.if.in.errors", netIf.InErrors, iface)
        ret[idx*20+3] = CounterValue("net.if.in.dropped", netIf.InDropped, iface)
        ret[idx*20+4] = CounterValue("net.if.in.fifo.errs", netIf.InFifoErrs, iface)
        ret[idx*20+5] = CounterValue("net.if.in.frame.errs", netIf.InFrameErrs, iface)
        ret[idx*20+6] = CounterValue("net.if.in.compressed", netIf.InCompressed, iface)
        ret[idx*20+7] = CounterValue("net.if.in.multicast", netIf.InMulticast, iface)
        ret[idx*20+8] = CounterValue("net.if.out.bytes", netIf.OutBytes, iface)
        ret[idx*20+9] = CounterValue("net.if.out.packets", netIf.OutPackages, iface)
        ret[idx*20+10] = CounterValue("net.if.out.errors", netIf.OutErrors, iface)
        ret[idx*20+11] = CounterValue("net.if.out.dropped", netIf.OutDropped, iface)
        ret[idx*20+12] = CounterValue("net.if.out.fifo.errs", netIf.OutFifoErrs, iface)
        ret[idx*20+13] = CounterValue("net.if.out.collisions", netIf.OutCollisions, iface)
        ret[idx*20+14] = CounterValue("net.if.out.carrier.errs", netIf.OutCarrierErrs, iface)
        ret[idx*20+15] = CounterValue("net.if.out.compressed", netIf.OutCompressed, iface)
        ret[idx*20+16] = CounterValue("net.if.total.bytes", netIf.TotalBytes, iface)
        ret[idx*20+17] = CounterValue("net.if.total.packets", netIf.TotalPackages, iface)
        ret[idx*20+18] = CounterValue("net.if.total.errors", netIf.TotalErrors, iface)
        ret[idx*20+19] = CounterValue("net.if.total.dropped", netIf.TotalDropped, iface)
    }
    return ret
}
```
For example, we may directly change Bit into Byte and modify the name of counter at the same time when the agent reports the data, as follows:
```
ret[idx*20+0] = CounterValue("net.if.in.bits", netIf.InBytes*8, iface)
```