# docker-rtorrent

This is a minimal alpine container that includes rtorrent only. It uses multi
stage builds to build the image, so build dependencies are not included in the
final image.

A built version of master can be pulled from `munnerz/rtorrent:latest`
