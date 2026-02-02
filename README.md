/**
 * Modified Whatsapp-API
 * Bail By Rixzz – FIXED VERSION
 */

const {
  default: makeWASocket,
  fetchLatestBaileysVersion,
  useMultiFileAuthState,
  DisconnectReason
} = require('@whiskeysockets/baileys')

const fs = require('fs')
const P = require('pino')

/* =======================
   CONNECT (QR / PAIRING)
======================= */

async function connectWA({ pairing = false, number = null }) {
  const { state, saveCreds } = await useMultiFileAuthState('./session')
  const { version } = await fetchLatestBaileysVersion()

  const client = makeWASocket({
    auth: state,
    logger: P({ level: 'silent' }),
    printQRInTerminal: !pairing,
    browser: ['Ubuntu', 'Chrome', '20.0.1'],
    version
  })

  if (pairing && number && !client.authState.creds.registered) {
    const code = await client.requestPairingCode(number.trim())
    console.log('Your pairing code:', code)
  }

  client.ev.on('creds.update', saveCreds)

  client.ev.on('connection.update', (update) => {
    const { connection, lastDisconnect } = update
    if (connection === 'close') {
      if (
        lastDisconnect?.error?.output?.statusCode !==
        DisconnectReason.loggedOut
      ) {
        connectWA({ pairing, number })
      }
    } else if (connection === 'open') {
      console.log('WhatsApp Connected')
    }
  })

  return client
}

/* =======================
   SEND ORDER MESSAGE
======================= */

async function sendOrderMessage(client, m) {
  const thumb = fs.readFileSync('./ZeppImage.jpg')

  await client.sendMessage(
    m.chat,
    {
      orderMessage: {
        itemCount: 1,
        status: 1,
        surface: 1,
        message: "Gotta get a grip",
        orderTitle: "PongKangColi",
        sellerJid: m.chat,
        thumbnail: thumb,
        totalAmount1000: 72502,
        totalCurrencyCode: "IDR"
      }
    },
    { quoted: m }
  )
}

/* =======================
   SEND POLL RESULT SNAPSHOT
======================= */

async function sendPollResult(client, m) {
  await client.sendMessage(m.chat, {
    pollResultSnapshotMessage: {
      name: "n",
      options: [
        { optionName: "poll 1", voteCount: 10 },
        { optionName: "poll 2", voteCount: 5 }
      ],
      totalVotes: 15
    }
  })
}

/* =======================
   SEND PRODUCT MESSAGE
======================= */

async function sendProductMessage(client, m) {
  const thumb = fs.readFileSync('./ZeppImage.jpg')

  await client.relayMessage(
    m.chat,
    {
      productMessage: {
        product: {
          productImage: {
            mimetype: "image/jpeg",
            jpegThumbnail: thumb
          },
          title: "DewaBail",
          description: "zZZ...",
          priceAmount1000: 72502,
          currencyCode: "IDR",
          retailerId: "EXAMPLE_RETAILER_ID",
          productId: "EXAMPLE_TOKEN",
          url: "https://t.me/RixzzNotDev"
        },
        businessOwnerJid: m.chat,
        body: "Nak Tido",
        footer: "Footer",
        buttons: [
          {
            buttonId: "cta_url",
            buttonText: { displayText: "7eppeli-Pdf" },
            type: 1
          }
        ]
      }
    },
    {}
  )
}

/* =======================
   EXPORT
======================= */

module.exports = {
  connectWA,
  sendOrderMessage,
  sendPollResult,
  sendProductMessage
}
