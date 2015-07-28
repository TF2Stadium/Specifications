# Introduction

We’re using Go, GORM (with postgres), socket.io and gorilla for the backend. The client
and the server backend communicate using socket.io, which allows us to have
a request-acknowledgement architecture.

For the frontend, we’re using AngularJS and the Foundation Framework.

For communicating with the game server, we have developed our own TF2 RCON
Wrapper around a [go rcon](https://github.com/james4k/rcon) library. Mumble channels are managed used
[Gumble](https://github.com/layeh/gumble), a Go client library for Mumble.
