# Self-hosting Sites with Linode and Caddy

*Apr 4, 2022*

*Draft*

After I put up my site again but instead of hosting it on GitHub
Pages I decided to self-host it.

I created a 1GB Nanode on [Linode](https://www.linode.com/?r=1fef70abe36a3f9c7202335d3d697dd5e3c99fb9), installed [Caddy](https://caddyserver.com/)

- dns config

```bash
$ cat Caddyfile
www.mirovarga.com {
        redir https://mirovarga.com{uri}
}

mirovarga.com {
        root * /home/mirko/mirovarga.com/static
        file_server
        encode gzip 
}
```

Then start `caddy` in the backgroung so it doesn't stop after logout:

```bash
$ sudo caddy start
```
