<template>
    <div id="app">
        <img src="/ava_logo.png" class="logo">
        <h1>AVA Quicksign</h1>
        <div v-if="result" class="result">
            <label>Signed Message</label>
            <p class="signed_msg">{{result}}</p>
            <p class="explain">Please copy this signed message into the <a href="https://forms.gle/voK2jayGBCd7PNeg7" target="_blank">Incentivized Testnet Claims Form</a>
                in the Signature field. </p>
            <p>Questions? Reach us by email <a href="mailto:ava@avalabs.org">ava@avalabs.org</a> or Discord <a href="https://chat.avalabs.org/" target="_blank">https://chat.avalabs.org</a>.</p>
            <button @click="startAgain">Sign Again</button>
        </div>
        <div v-else class="main_page">
            <p>Enter your private key for the destination address associated with your nodeID. The call to create this is seen below. The address indicated has a private key associated with it. This is the private key that should be entered into the field below.</p>
            <img src="/destination.png" class="cover">
            <p>If you do not know this private key, invoke the
                <span class="code">platform.exportKey</span>
                (<a href="https://docs.ava.network/v1.0/en/api/platform/#platformexportkey" target="_blank">view in docs</a>)
                call on your running node.
            </p>
            <p><a href="https://medium.com/avalabs/how-to-claim-incentivized-testnet-rewards-f96b7ee3bd2d" target="_blank">Full Instructions</a></p>

            <form @submit.prevent="sign">
                <label>Private Key</label>
                <input type="text" v-model="pk">
                <p v-if="isErr" class="err">Invalid private key</p>
                <button>Sign</button>
            </form>
            
            <p>Questions? Reach us by email <a href="mailto:ava@avalabs.org">ava@avalabs.org</a> or Discord <a href="https://chat.avalabs.org/" target="_blank">https://chat.avalabs.org</a>.</p>
            
            <p><a href="https://github.com/ava-labs/ava-quicksign" target="_blank">Source Code</a></p>
        </div>


    </div>
</template>

<script>
    import * as slopes from "slopes"

    const MESSAGE = "I am the very model of a modern major general.";
    // import { Buffer } from 'buffer/'; // the slash forces this library over native Node.js Buffer
    let myNetworkID = 12345; //default is 2, we want to override that for our local network
    let myBlockchainID = "GJABrZ9A6UQFpwjPU8MDxDd8vuyRoDVeDAXc694wJ5t3zEkhU"; // The AVM blockchainID on this network
    let ava = new slopes.Slopes("localhost", 9650, "http", myNetworkID, myBlockchainID);
    let avm = ava.AVM(); //returns a reference to the AVM API used by Slopes
    let keychain = avm.keyChain();

    let bintools = slopes.BinTools.getInstance();
    // let crypto = new slopes.CryptoHelpers();

    console.log()


    export default {
        name: 'App',
        data(){
            return {
                pk: "",
                isErr: false,
                result: null,
            }
        },
        created() {
            console.log(slopes)
        },
        methods: {
            startAgain(){
                this.pk = '';
                this.result = null;
                this.isErr = false;
            },

            sign(){
                this.isErr = false;
                let pk = this.pk;
                try{
                    let mypk = bintools.avaDeserialize(pk); //returns a Buffer
                    let addr = keychain.importKey(mypk); //returns a Buffer for the address
                    let keypair = keychain.getKey(addr);


                    // let msgBuff = Buffer.from(crypto.sha256(MESSAGE));


                    // Sign the message with the key
                    // let msgBuff = bintools.stringToBuffer(MESSAGE);
                    let ch = new slopes.CryptoHelpers();
                    let msgBuff =Buffer.from(ch.sha256(MESSAGE), 'utf8');
                    let signedBuff = keypair.sign(msgBuff);

                    let isCorrect = keypair.verify(ch.sha256(MESSAGE), signedBuff);

                    let pubKey = keypair.recover(ch.sha256(MESSAGE), signedBuff);
                    let addressBuff = keypair.addressFromPublicKey(pubKey);
                    let address = bintools.avaSerialize(addressBuff);
                    console.log("isCorrect", isCorrect);
                    console.log("address", address, bintools.avaSerialize(addr));

                    let res = bintools.avaSerialize(signedBuff);


                    this.result = res;
                }catch(e){
                    this.isErr = true;
                    console.log(e);
                }
            }
        }
    }
</script>

<style lang="scss">
    html, body, #app{
        min-height: 100%;
        margin: 0;
    }
    p{
        margin: 15px 0px;
        word-break: break-word;
    }
    #app {
        font-family: Avenir, Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        /*text-align: center;*/
        color: #2c3e50;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        padding: 8px 14px;
        /*padding-top: 25vh;*/
    }

    .main_page, .result{
        width: 460px;
        max-width: 100%;
        justify-content: center;
    }

    span.code{
        background-color: #d2d2d2;
        color: #000;
        padding: 3px 12px;
        border-radius: 4px;
        font-size: 13px;
    }

    form, .result{
        margin-top: 30px;

        > *{
            width: 100%;
        }
    }

    input{
        display: block;
        box-sizing: border-box;
        outline: none;
        border-color: #999;
        border: none;
        background-color: #f5f6fa;
        padding: 12px 14px;
        text-align: center;
    }

    label{
        display: block;
        text-align: left;
        font-size: 12px;
        color: #999;
        width: 100%;
    }

    img{
        object-fit: contain;
    }


    .logo{
        width: 80px;
    }
    .cover{
        width: 100%;
    }

    button{
        margin: 14px 0px;
        background-color: #e84142;
        padding: 8px 8px;
        color: #fff;
        border-radius: 2px;
        border: none;
        font-size: 14px;
        text-align: center;
        outline: none;
        cursor: pointer;
    }

    .signed_msg{
        word-break: break-all;
        text-align: left;
        background-color: #f2f2f2;
        border: 1px solid #ddd;
        padding: 8px 14px;
        border-radius: 4px;
        margin-bottom: 15px;
        font-size: 12px;
    }

    .err{
        text-align: left;
        color: #f44;
    }

    .explain{
        text-align: left;
    }
</style>
