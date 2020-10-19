After clone submodule need to be updated:  
`git submodule update --recursive --init --remote`

Build website:  
`docker run --rm -it -v $(pwd):/src -p 1313:1313 klakegg/hugo:ext`

Run in nginx for testing:  
`docker run --rm -it --name blog-nginx -v $(pwd)/public:/usr/share/nginx/html -p 1313:80 nginx`

Create new page:  
`docker run --rm -it -v $(pwd):/src klakegg/hugo:ext new post/start-with-hugo.md`
