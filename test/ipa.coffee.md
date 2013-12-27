    mocha = require 'mocha'
    should = require 'should'

    {EventEmitter} = require 'events'

    describe 'When using IPA', ->
      ipaProtocol = require '..'
      ipaccessProtocol = (require 'ipaccess').protocol
      ipaccessHandler = (require 'ipaccess').handler
      describe 'the protocol handler', ->
        it 'should respond with pong to a ping', (done) ->
          conn = new EventEmitter
          ipa = new ipaProtocol conn
          ipaccess = new ipaccessProtocol ipa
          handler = new ipaccessHandler ipaccess
          conn.write = (buf) ->
            buf.should.have.length 4
            buf.toJSON().should.eql [0x00,0x01,0xfe,0x01]
            done()
          conn.emit 'data', new Buffer [ 0x00, 0x01, 0xfe, 0x00 ]
