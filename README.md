# netfix-list
This is a list of domains that have regional problem with connections.
I use this list to gen Mikrotik DNS static list for reroute this domains to another route.
Address list needs more DNS chache space so we need extend this space:
```
/ip dns set cache-size=20000KiB
```
Next script for Mikrotik combine this list with antifilter.download community list and import in Mikrotik (change **listname** on what you need, in example "to-wg"):
```
:do {
    :local listname "to-wg";
    :local url ("https://legionlh.ru/dnslist.php?list=".$listname);

    :do {
        /file remove "/dnslist.rsc";
    } on-error={}

    :put "Downloading dnslist.rsc...";
    :do {
    /tool fetch url=$url dst-path="/dnslist.rsc"
    } on-error={
        :put "Error. Download failed";
    }
    
    /ip dns static remove [find where address-list=$listname]    
    
    :put "Importing dnslist.rsc...";
    :do {
        /import "/dnslist.rsc";
    } on-error={
        :put "import failed. unknown error.";
    }

    :put "Update Complete.";

} on-error={};
```
To forward this list to another route (change dst-address-list on what you need, in example "to-wg", and gateway on your alternative gateway, in example "172.30.31.1"):
```
/ip firewall mangle add action=mark-routing chain=prerouting dst-address-list=to-wg new-routing-mark=to-wg-mark
/ip route add disabled=no distance=11 dst-address=0.0.0.0/0 gateway=172.30.31.1 routing-table=to-wg-mark scope=30 target-scope=10
```
**(not required)** To avoid differences between the client's and the router's DNS, it is recommended to intercept all DNS requests in your local network. This can't do anything with DoH so prefer to disable DoH in clients browsers.
```
/ip firewall nat
add action=redirect chain=dstnat dst-port=53 in-interface-list=LAN protocol=udp
add action=redirect chain=dstnat dst-port=53 in-interface-list=LAN protocol=tcp
```
