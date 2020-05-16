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

echo "get 10.1.2.14-expand" | redis-cli
"70461da8b94c6ca5d2fda3260c5a8c3b,Down,RED,prod,404"
