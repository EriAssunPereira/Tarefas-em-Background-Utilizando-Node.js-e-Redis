# Tarefas-em-Background-Utilizando-Node.js-e-Redis

Implementando um sistema de cadastro de usuário com envio de e-mail de confirmação como uma tarefa em background utilizando Node.js e Redis.

---

## Introdução

No desenvolvimento de aplicações web modernas, é comum a necessidade de executar tarefas que são intensivas em termos de tempo e recursos, como o envio de e-mails. Para não bloquear o thread principal do Node.js e oferecer uma resposta rápida ao usuário, essas tarefas podem ser executadas em background. Neste artigo, vamos explorar como implementar esse processo utilizando Node.js e Redis.

## Pré-requisitos

Para seguir este tutorial, você precisará ter o Node.js e o Redis instalados em seu ambiente de desenvolvimento. Além disso, conhecimentos básicos de JavaScript e experiência com o Express.js serão úteis.

## Configuração do Projeto

Primeiro, vamos configurar um novo projeto Node.js e instalar as dependências necessárias:

```bash
mkdir user-registration
cd user-registration
npm init -y
npm install express nodemailer bull redis dotenv
```

Aqui, `express` é o framework web, `nodemailer` é um módulo para enviar e-mails, `bull` é uma biblioteca de filas para gerenciar tarefas em background, e `redis` é o cliente para interagir com o Redis.

## Estrutura do Projeto

Vamos organizar nosso projeto em módulos para manter o código limpo e manutenível:

```
user-registration/
|-- node_modules/
|-- src/
|   |-- config/
|   |   |-- bullConfig.js
|   |   |-- mailConfig.js
|   |-- jobs/
|   |   |-- emailJob.js
|   |-- queues/
|   |   |-- emailQueue.js
|   |-- server.js
|-- package.json
```

## Configuração do Bull e Redis

No arquivo `src/config/bullConfig.js`, vamos configurar o Bull para conectar-se ao Redis:

```javascript
const Queue = require('bull');
const redisConfig = {
  redis: {
    host: process.env.REDIS_HOST,
    port: process.env.REDIS_PORT,
    password: process.env.REDIS_PASS,
  },
};

const emailQueue = new Queue('email', redisConfig);

module.exports = emailQueue;
```

## Envio de E-mail

No diretório `src/jobs/`, vamos criar o arquivo `emailJob.js` que conterá a lógica para enviar e-mails:

```javascript
const nodemailer = require('nodemailer');
const mailConfig = require('../config/mailConfig');

const transporter = nodemailer.createTransport(mailConfig);

const sendConfirmationEmail = async (user) => {
  const mailOptions = {
    from: 'no-reply@webloggerbrasil.com',
    to: user.email,
    subject: 'Confirmação de Cadastro',
    text: `Olá, ${user.name}! Confirme seu cadastro clicando no link: ${user.confirmationLink}`,
  };

  await transporter.sendMail(mailOptions);
};

module.exports = sendConfirmationEmail;
```

## Filas de Tarefas

No diretório `src/queues/`, vamos criar o arquivo `emailQueue.js` para adicionar tarefas à fila:

```javascript
const emailQueue = require('../config/bullConfig');
const sendConfirmationEmail = require('../jobs/emailJob');

const addEmailToQueue = (user) => {
  emailQueue.add({
    user,
  });
};

emailQueue.process(async (job, done) => {
  await sendConfirmationEmail(job.data.user);
  done();
});

module.exports = addEmailToQueue;
```

## Servidor e Rotas

Finalmente, no arquivo `src/server.js`, vamos configurar o servidor Express e a rota de cadastro de usuário:

```javascript
require('dotenv').config();
const express = require('express');
const addEmailToQueue = require('./queues/emailQueue');

const app = express();

app.use(express.json());

app.post('/register', (req, res) => {
  const { name, email } = req.body;
  // Aqui você adicionaria o usuário ao banco de dados e geraria um link de confirmação
  const confirmationLink = 'http://webloggerbrasil.com/confirm';

  addEmailToQueue({ name, email, confirmationLink });

  res.status(200).send('Usuário cadastrado com sucesso!');
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Servidor rodando na porta ${PORT}`));
```

## Conclusão

Com este sistema, você pode registrar usuários e enviar e-mails de confirmação sem bloquear o thread principal do Node.js. O uso do Redis com o Bull permite gerenciar as tarefas em background de forma eficiente e escalável.

---

Espero que este artigo tenha sido útil para entender como implementar tarefas em background utilizando Node.js e Redis. Lembre-se de substituir as variáveis de ambiente e os dados fictícios pelos reais em seu projeto.
