IP Access Connection
====================

    {EventEmitter} = require 'events'

IP Access Connection
--------------------

TCP Connection to a single BTS is handled here by unframing the TCP stream.

    module.exports = class ipaProtocol extends EventEmitter

      constructor: (@connection) ->

        buffered = new Buffer 0

        @connection.on 'error', (err) =>
          console.error "Socket error #{err}"

When the far-end (i.e. BTS) disconnects,

        @connection.on 'end', =>
          console.error 'Socket disconnected'

flush any leftover frame.

          @framer buffered

        @connection.on 'data', (buf) =>
          buffered = Buffer.concat [buffered, buf]

          buffered = @framer buffered

Take a protocol frame (with protocol marker as first octet) and send it out.

      send: (proto,frame,cb) ->
        buf = new Buffer frame.length + 3
        buf.writeUInt16BE frame.length, 0
        if typeof proto is 'string'
          proto = @proto[proto]
        buf.writeUInt8 proto, 2
        frame.copy buf, 3
        @connection.write buf, cb

Take a buffer and splice it into protocol frames (with protocol marker as first octect).

      framer: (buf) ->
        if buf.length < 2
          return buf

The first two octets of an IP.Access frame contain a network-byte-order length of the data (the protocol designator is not included).
The protocol designator is in the third octet.

        ipa_len = buf.readUInt16BE 0
        frame_end = 3+ipa_len
        if buf.len < frame_end
          return buf

The parser receives the frame with the protocol designator at offset zero.

        @parser buf.slice 2, frame_end

        return buf.slice frame_end

Ref: libosmo-abis/include/osmocom/abis/ipaccess.h

      proto:
        rsl:       0x00
        ip_access: 0xfe
        sccp:      0xfd
        oml:       0xff
        osmo:      0xee
        mgcp_old:  0xfc

      proto_message:
        0x00: 'rsl'
        0xfe: 'ip_access'
        0xfd: 'sccp'
        0xff: 'oml'
        # OpenBSC extensions
        0xee: 'osmo'
        0xfc: 'mgcp_old'

      parser: (frame) ->
        proto = frame[0]

        data = frame.slice 1

        if @proto_message[proto]?
          # console.log "Sending `protocol #{@proto_message[proto]}` to any ipaProtocol consumer."
          @emit "protocol #{@proto_message[proto]}", data
        else
          console.error "Unknown protocol #{proto}, framing error?"
