# change-watcher
Store response hashes in redis, curate response data. Trigger checks on change.

Current version is developed for running on FreeBSD, but I'm using the approach of try BSD
variation first, then fail back to linux variation. 

You can curate or generate another file 

expand.csv

that has expanded lookups for specific hash values.

Example expand.csv

198d798fc0477d92698dc43c416bf460,Healthy,GREEN,cust,200
36227e18e469d3e9718550d7430f425a,MaintenancePage,YELLOW,cust,200
70461da8b94c6ca5d2fda3260c5a8c3b,Down,RED,cust,404
fdc34979246459e973c5db2023f83aeb,Down,RED,cust,DNS_RESOLVE_FAILURE

where that md5 value is the result of

curl https://yourmonitoring-target | md5

or on linux

curl https://yourmonitoring-target | md5sum

So you can either exactly emulate the data of an error state to find the hash to use in expand.csv
or collect them as you go and add them in. You don't have to populate expand.csv unless you want to.

And if you don't want to use md5 for the hashing, no big deal, just replace md5 with sha512 or whatever you like in the scripts.

Alright, so then you can have other apps fetch the status, example:

echo "get 10.1.2.14-expand" | redis-netloop:5843
"70461da8b94c6ca5d2fda3260c5a8c3b,Down,RED,prod,404"

Or for the data without the expand.csv data:

echo "get 10.1.2.13" | redis-netloop:5843
"10.1.2.13,70461da8b94c6ca5d2fda3260c5a8c3b"

Redis is so high performance that we can have many front ends and systems read from it continually, while the
backend is gently performing a single set of health checks. This helps with network scaling to reduce reduntant
monitoring or health checking. You can have thousands of nodes query the redis database, which can handle it,
instad of having thousands of nodes query your production endpoint etc. 
If you have multiple datacenters involved, you would run a change-watcher instance in each data center and have
systems in that same datacenter query from the local one. Otherwise, if route from DC A is up and DC B is down,
but you only put change-watcher in DC B, then the monitoring will show all down even though DC A is up.

When the response changes, so will the hash, this includes lack of a response or any kind of error.
Learn those hash responses to the different conditions, map them to your expand.csv


If you want to change the curl to do a POST for an API etc, easy enough to add in :)

When a change happens, we trigger the recon-proc that will do analysis on:

- local gateway ICMP
- debug TLS interaction with targets
- target ICMP



Do not pass URI contexts in like /something or thing/something, the TLS iteraction will fail because it
uses openssl s_client -connect $value:443 which does not use URI context before the :443

Example crontab

0-59 * * * * /usr/local/bin/chkwatch myapi.com emailserver-1 parter-server-4.partnerplace.com 10.2.1.13 10.3.1.13
