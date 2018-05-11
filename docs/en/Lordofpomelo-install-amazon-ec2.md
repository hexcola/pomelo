I finally managed to get 'Lords of Pomelo' working in the Amazon EC2 infrastructure on a Ubuntu 12.04 server instance.

I'd like to share some insights on a couple of challenges I faced.

- the smallest of the Amazon virtual machines (t1.micro) just won't cut it. If you try, you will find out that the various processes kill your mysql server. You need at least an m1.small and this will also be not big enough for any real world testing.
- getting the installation to work took me some time, because things fail silently sometimes (or get buried in the logs). Namely this goes for the 'pidstat' (in the sysstat package) which means needs to be installed. I added this to the wiki.
- don't try to kill the pomelo processes by ctrl-c, because they will spawn again (why?), and things will get uncontrollable. Use 'pomelo kill --force' always.
- other problems were related to ports that I didn't open by default. I still have to find which ones you really need, but it's safe to say that if you open 3000-3100 you are good to go (but you probably need only three open ports)

I wrote one big shell script that should get you up and running on EC2:
https://gist.github.com/dirkk0/7672152

I continue reporting my findings here.

By Dirk Krause 