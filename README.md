# escola-futebol-admin
escola de futebol
// Estrutura simplificada para o backend e frontend do sistema admin de escola de futebol

// === BACKEND (Node.js + Express) ===
// backend/server.js
const express = require('express');
const cors = require('cors');
const axios = require('axios');
require('dotenv').config();

const app = express();
app.use(cors());
app.use(express.json());

app.post('/api/boletos', async (req, res) => {
  const { nome, email, cpfCnpj, valor, vencimento } = req.body;

  try {
    const cliente = await axios.post(
      'https://www.asaas.com/api/v3/customers',
      { name: nome, email, cpfCnpj },
      { headers: { Authorization: `Bearer ${process.env.ASAAS_TOKEN}` } }
    );

    const customerId = cliente.data.id;

    const boleto = await axios.post(
      'https://www.asaas.com/api/v3/payments',
      {
        customer: customerId,
        billingType: 'BOLETO',
        value: valor,
        dueDate: vencimento,
      },
      { headers: { Authorization: `Bearer ${process.env.ASAAS_TOKEN}` } }
    );

    res.json(boleto.data);
  } catch (error) {
    res.status(500).json({ erro: error.message });
  }
});

app.listen(3001, () => console.log('Backend rodando na porta 3001'));


// backend/.env.example
AS AAS_TOKEN=sua_chave_da_asaas_aqui


// === FRONTEND (React com Vite) ===
// frontend/src/pages/Login.jsx
import { useState } from 'react';

export default function Login() {
  const [email, setEmail] = useState('');
  const [senha, setSenha] = useState('');

  const handleLogin = () => {
    alert(`Email: ${email}\nSenha: ${senha}`);
  };

  return (
    <div style={{ padding: '2rem' }}>
      <h2>Login do Administrador</h2>
      <input placeholder="Email" onChange={(e) => setEmail(e.target.value)} /><br /><br />
      <input type="password" placeholder="Senha" onChange={(e) => setSenha(e.target.value)} /><br /><br />
      <button onClick={handleLogin}>Entrar</button>
    </div>
  );
}


// frontend/src/supabase.js
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = 'https://seu-projeto.supabase.co';
const supabaseKey = 'sua-chave-anon';
export const supabase = createClient(supabaseUrl, supabaseKey);


// frontend/src/main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import Login from './pages/Login';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <Login />
  </React.StrictMode>
);


// frontend/index.html
<!DOCTYPE html>
<html lang="pt-BR">
  <head>
    <meta charset="UTF-8" />
    <title>Escola de Futebol Admin</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>


// frontend/vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
});
