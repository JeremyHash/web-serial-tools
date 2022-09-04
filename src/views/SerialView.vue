<template>
  <v-main>
    <v-btn class="mt-n6 mx-2" x-large color="primary" @click="findPort">
      {{ findPortBtnText }}
    </v-btn>
    <v-btn class="mt-n6 mx-2" x-large color="primary" @click="clearData">
      {{ clearDataArea }}
    </v-btn>
    <v-select v-model="baudRate" dense style="width:150px" class="d-inline-flex mt-3 mx-1" :items="items" label="波特率"
      filled>
    </v-select>
    <v-checkbox class="d-inline-flex mx-2" v-model="hexDisply" label="Hex显示"></v-checkbox>
    <v-checkbox class="d-inline-flex mx-2" v-model="hexSend" label="Hex发送"></v-checkbox>
    <v-checkbox class="d-inline-flex mx-2" v-model="newLine" label="回车换行"></v-checkbox>
    <v-text-field dense class="d-inline-flex mx-2" v-model="sendData" filled clearable></v-text-field>
    <v-btn :disabled="sendStatus" class="mt-n6 mx-2" x-large color="primary" @click="send"> 发送
    </v-btn>
    <textarea readonly
      style="border: 1px solid black; display:block;width: 100%;height: 85%;resize: none;padding: 0px 10px;"
      id="serialDataArea" height="515">
    </textarea>
  </v-main>
</template>

<script>
import { tr } from 'vuetify/lib/locale'

export default {
  name: "SerialView",
  data() {
    return {
      serialDataArea: null,
      textAreaHeight: `${parseInt(document.documentElement.clientHeight) - 197}`,
      findPortBtnText: "寻找端口",
      clearDataArea: "清空数据",
      baudRate: "115200",
      hexDisply: false,
      hexSend: false,
      newLine: true,
      port: null,
      sendData: null,
      receiveData: "",
      sendStatus: true,
      textDecoder: null,
      textEncoder: null,
      readableStreamClosed: null,
      writableStreamClosed: null,
      reader: null,
      writer: null,
      items: ['9600', '115200', '256000', '460800', "921600", "1500000"],
    }
  },
  methods: {
    findPort: async function () {
      if (this.port !== null) {
        this.reader.cancel()
        await this.readableStreamClosed.catch((e) => { console.log(e) })
        this.writer.close()
        await this.writableStreamClosed
        await this.port.close()
        this.findPortBtnText = "寻找端口"
        this.port = null
        this.sendStatus = true
      } else {
        this.port = await navigator.serial.requestPort()
        await this.port.open({ baudRate: parseInt(this.baudRate) })
        navigator.serial.addEventListener("connect", (event) => {
          // TODO: 自动打开事件。目标器或警告用户端口可用。
          console.log(event)
        })

        navigator.serial.addEventListener("disconnect", (event) => {
          // TODO: Remove |event.target| from the UI.
          // 如果打开了串行端口，还会观察到流错误。
          console.log(event)
        })
        this.findPortBtnText = "关闭端口"
        this.sendStatus = false
        this.textDecoder = new TextDecoderStream()
        this.readableStreamClosed = this.port.readable.pipeTo(this.textDecoder.writable)
        this.reader = this.textDecoder.readable.getReader()
        this.textEncoder = new TextEncoderStream()
        this.writableStreamClosed = this.textEncoder.readable.pipeTo(this.port.writable)
        this.writer = this.textEncoder.writable.getWriter()

        // await writer.write("hello")
        while (true) {
          const { value, done } = await this.reader.read()
          if (done) {
            this.reader.releaseLock()
            break
          }
          if (this.hexDisply === true) {
            serialDataArea.value += this.stringtoHex(value)
          } else {
            serialDataArea.value += value
          }
          serialDataArea.scrollTop = serialDataArea.scrollHeight
        }
      }
    },
    clearData: function () {
      this.serialDataArea.value = ""
    },
    send: async function () {
      if (this.sendData === null || this.sendData === "") {
        return
      } else {
        if (this.newLine === true) {
          if (this.hexSend === true) {
            await this.writer.write(`${this.hextoString(this.sendData)}\r\n`)
          } else {
            await this.writer.write(`${this.sendData}\r\n`)
          }
        } else {
          if (this.hexSend === true) {
            await this.writer.write(`${this.hextoString(this.sendData)}`)
          } else {
            await this.writer.write(this.sendData)
          }
        }
      }
    },
    hextoString: function (hex) {
      var arr = hex.split("")
      var out = ""
      for (var i = 0; i < arr.length / 2; i++) {
        var tmp = "0x" + arr[i * 2] + arr[i * 2 + 1]
        var charValue = String.fromCharCode(tmp)
        out += charValue
      }
      return out
    },
    stringtoHex: function (str) {
      var val = ""
      for (var i = 0; i < str.length; i++) {
        let res = str.charCodeAt(i).toString(16)
        if (res.length === 1) { res = `0${res}` }
        val += `${res}  `
      }
      return val
    }
  },
  mounted() {
    window.Jeremy = this
    this.serialDataArea = document.getElementById('serialDataArea')
  }
}
</script>
