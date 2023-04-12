# Intro

This library allows you to establish p2p connections between polkadot accounts and instantly exchange data in both directions.

# Usage:

To work with DOTRTC, you first need to create a keyring, under which the account will connect to other accounts:

    import {Keyring} from '@polkadot/api';
    const accountKeyring = (new Keyring({type: 'sr25519'})).addFromUri(`<secret phrase>`);  //keyring of your wallet

Next, you can create a DOTRTC instance with the necessary settings and specify event handlers (connection request, successful connection, disconnection):

    const p2p = new DOTRTC({
        iceServer:      string,                 //Stun or turn server (https://datatracker.ietf.org/doc/html/rfc8445), example: `stun:stun.services.mozilla.com`
        chunkSize:      integer                 //All packets sent will be split into small packets of this size in bytes (example 65535)
                                                //This is necessary so as not to clog the entire channel when sending a large amount of data.
        endpoint:       string                  //Endpoint of parachain node, for example `wss://diffy.bsn.si`
        keyring: accountKeyring,                //Keyring of your wallet
        onConnectionRequest: function() {...}   //Handler that will be called when someone tries to connect to you
        onConnect: function() {...}             //Handler that will be called when a connection is successfully established with someone
        onDisconnect: function() {...}          //Handler will be called when the connection is broken (the remote user forcibly disconnected, or may be caused by problems with the Internet connection)
    });


When creating a DOTRTC instance, you must specify a successful connection handler:

    onConnect: function(channel) {
        console.log('connection established with:', channel.remoteAddress);
    }

Create a p2p connection to the account by his DOT address:

    p2p.connect({
        to: '<DOT ADDRESS>'
    });

After that, the onConnectionRequest event will fire for the user they are trying to connect to, in which a connection request will come.
The user will have to accept it or ignore it.

    onConnectionRequest: function(connection) {
        console.log(connection.remoteAddress);      //`connection.remoteAddress` contains the address of the user who is trying to connect to us
        connection.accept();                        //allow this user to connect to us
    }


Upon successful connection, the `onConnect` handler will be called, in which the `channel` object of the connection will come in the arguments, through which messages can be sended
The `channel` object has the following methods:

    channel.sendMessage(payload);                   //send message to remote address. Payload must be type Uint8Array
    channel.onMessage(payload => {                  //payload is Uint8Array
        console.log(data);
    });