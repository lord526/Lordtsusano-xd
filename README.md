js
const { default: makeWASocket, useSingleFileAuthState } = require('@whiskeysockets/baileys');
const { Boom } = require('@hapi/boom');
const { state, saveState } = useSingleFileAuthState('./auth.json');

async function startBot() {
  const sock = makeWASocket({
    auth: state,
    printQRInTerminal: true,
  });

  sock.ev.on('creds.update', saveState);

  sock.ev.on('messages.upsert', async ({ messages }) => {
    const msg = messages[0];
    if (!msg.message) return;

    const texte = msg.message.conversation || msg.message.extendedTextMessage?.text || '';
    const jid = msg.key.remoteJid;

    // Commandes simples
    if (texte === '!ping') {
      await sock.sendMessage(jid, { text: 'pong 🏓' });
    }

    if (texte === '!aide') {
      await sock.sendMessage(jid, {
        text: '📜 Commandes disponibles :\n!ping - Test\n!date - Heure actuelle',
      });
    }

    if (texte === '!date') {
      await sock.sendMessage(jid, {
        text: `🕒 Date et heure : ${new Date().toLocaleString()}`,
      });
    }
  });
}

startBot();
