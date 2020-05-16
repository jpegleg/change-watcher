# redis-change-watcher
Store response hashes in redis, curate response data. Trigger checks on change.

Current version is developed for running on FreeBSD, but I'm using the approach of try BSD
variation first, then fail back to linux variation. 

You can curate or generate another file 

expand.csv

that has expanded lookups for specific hash values.

Example expand.csv

198d798fc0477d92698dc43c416bf460,Healthy,GREEN,prod,200
36227e18e469d3e9718550d7430f425a,MaintenancePage,YELLOW,prod,200
70461da8b94c6ca5d2fda3260c5a8c3b,Down,RED,prod,404

where that md5 value is the result of

curl https://yourmonitoring-target/api/endpoint | md5

or on linux

curl https://yourmonitoring-target/api/endpoint | md5sum

And if you don't want to use md5 for the hashing, no big deal, just replace md5 with sha512 or whatever you like in the scripts.

Alright, so then you can have other apps fetch the status, example:

echo "get 10.1.2.14-expand" | redis-netloop:5843
"70461da8b94c6ca5d2fda3260c5a8c3b,Down,RED,prod,404"

When the response changes, so will the hash, this includes lack of a response or any kind of error.
Learn those hash responses to the different conditions, map them to your expand.csv
And boom, you will have deep, high performance, change monitoring.

If you want to change the curl to do a POST for an API etc, easy enough to add in :)

When a change happens, we trigger the recon-proc that will do analysis on:

- local gateway ICMP
- debug TLS interaction with targets
- target ICMP

Once the recon-proc gathers the data, it bundles it into a report with the original response hash and the expand.csv lookup.
If you have the file email.trigger in the work directory containing the email address to send to, it will email the reports when they arrive.
