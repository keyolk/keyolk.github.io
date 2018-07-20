+++
title  = "http"
toc    = true
weight = 0
+++

## HTTP/1.1
### Issue
- Head-of-line blocking, HoL
  - Single request / connection
  - Request must be responded in order, FIFO
- No optimised well
  - Uncompressed headers
  - Redundant headers
- Lack of Server-side push

## SPDY&HTTP2
- Multiplexed streams
  - Multiple stream over single TCP connection
- Request Prioritization
- HTTP header compression
  - HPACK format
  - Huffman code
- Server Push

## TCP/TLS
### Issue
- HoL
- TCP 3-way handshaking
- TLS negotiation handshaking
- Large backoff on packet loss
- Poor perforamnce in mobile

## QUIC
Quick UDP Internet Connections
