
[Source](http://stackoverflow.com/questions/18202485/how-to-test-the-result-of-applying-a-puppet-template-to-given-test-parameters "Test the result of a puppet template given test parameters")
jjjjj

```sh
#!/usr/bin/env ruby
require 'erb'
#Test Variables
jmx_port = 9200
@markets = ['CH', 'FR']

temp = File.open("testerb.erb", "rb").read;
renderer = ERB.new(temp)
puts output = renderer.result()
```

```ruby
{
  "servers" : [ {
    "port" : "<%= jmx_port %>",
    "host" : "localhost",

    "queries" : [
      <% @markets.each do |market| %>
    {
      "outputWriters" : [ {
        "@class" : "com.googlecode.jmxtrans.model.output.StdOutWriter",
      } ],
      "obj" : "solr/market_<%= market %>:type=queryResultCache,id=org.apache.solr.search.LRUCache",
      "attr" : [ "hits","hitratio"]
    },
    <% end %>
    ],
    "numQueryThreads" : 2
  } ]
}
```
