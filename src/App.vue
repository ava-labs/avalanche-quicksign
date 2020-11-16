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
                <label>Message</label>
                <input type="text" v-model="msg">
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
    import { Avalanche, BinTools, Buffer } from "avalanche";
    import createHash from 'create-hash';

    let ip = "localhost";
    let port = 9650;
    let protocol = "http";
    let networkID = 12345;
    let avalanche = new Avalanche(ip, port, protocol, networkID);
    let avm = avalanche.XChain(); //returns a reference to the AVM API used by AvalancheJS 
    let keychain = avm.keyChain();

    let bintools = BinTools.getInstance()

    console.log()


    export default {
        name: 'App',
        data(){
            return {
                pk: "",
                msg: "",
                isErr: false,
                result: null,
            }
        },
        created() {
        },
        methods: {
            startAgain(){
                this.pk = '';
                this.result = null;
                this.isErr = false;
            },

            sign(){
                this.isErr = false;
                let message = this.msg;
                let pk = this.pk;
                try{
                    let mypk = bintools.cb58Decode(pk); //returns a Buffer
                    let keypair = keychain.importKey(mypk); //returns a Buffer for the address
                    let addr = keypair.getAddress();

                    // Sign the message with the key
                    let buff = Buffer.from(message, "utf8");
                    let digest = createHash('sha256').update(buff).digest();
                    let signedBuff = keypair.sign(digest);

                    let isCorrect = keypair.verify(digest, signedBuff);

                    let pubKey = keypair.recover(digest, signedBuff);
                    let addressBuff = keypair.addressFromPublicKey(pubKey);
                    let address = bintools.cb58Encode(addressBuff);
                    console.log("isCorrect", isCorrect);
                    console.log("address", address, bintools.cb58Encode(addr));

                    let res = bintools.cb58Encode(signedBuff);


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
