
[https://github.com](https://github.com/evolvingweb/puppet-glusterfs/issues/2 "generate memory size for applications based upon facter")

```nix
irb> require 'facter'
irb> memorysize = 0
irb> Facter.each{ |k,v| memorysize = v if k =~ /memorysize/ }
irb> memorysize #=> "7.81 GB"
irb> (memorysize.to_i*0.2).floor #=> 1
irb> (memorysize.to_f*1024*0.2).floor #=> 1599
```
