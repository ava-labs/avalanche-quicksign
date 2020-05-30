<template>
    <div id="app">
        <div v-if="result" class="result">
            <label>Signed Message</label>
            <p class="signed_msg">{{result}}</p>
            <p class="explain">Please send this signed message to ava@ava.ava</p>
        </div>
        <form @submit.prevent="sign" v-else>
            <label>Private Key</label>
            <input type="text" v-model="pk">
            <p v-if="isErr" class="err">Invalid private key</p>
            <button>Sign</button>
        </form>

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
                    let msgBuff =Buffer.from(MESSAGE, 'utf8');
                    let signedBuff = keypair.sign(msgBuff);

                    let isCorrect = keypair.verify(MESSAGE, signedBuff);
                    console.log(isCorrect);

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
    p{
        margin: 4px 0px;
    }
    #app {
        font-family: Avenir, Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        margin-top: 60px;
        display: flex;
        justify-content: center;
    }

    form, .result{
        width: 320px;
        max-width: 100%;
        justify-content: center;
        margin-top: 33vh;

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

    button{
        margin: 14px 0px;
        background-color: #2960cd;
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
        margin-bottom: 15px;
    }

    .err{
        text-align: left;
        color: #f44;
    }

    .explain{
        text-align: left;
    }
</style>
