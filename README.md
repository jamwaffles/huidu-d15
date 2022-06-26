# Huidu D15

Notes and stuff around bringing up a Huidu D15 board. Mostly as a learning experience for Yocto, but
also with the potential to be the basis of a pretty capable CNC control board.

I'm using Claude Schwarz'
[Twitter thread](https://twitter.com/Claude1079/status/1513541029399519234) as
reference/inspiration.

## Serial

- "CoolTerm" seems to be an alright serial terminal for Windows.
- 115200 baud works for me with default software, although
  [apparently](https://twitter.com/Claude1079/status/1512693171305779202) 1.5MBaud (1500000) is
  default for Rockchips.
- Connections:

  ![](images/serial.jpeg)
