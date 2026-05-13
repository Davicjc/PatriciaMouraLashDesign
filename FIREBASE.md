# Como configurar o Firebase (backend do agendamento)

O site e o painel admin usam o **Firebase** do Google como banco de dados.
É gratuito para este volume de uso e não precisa de servidor próprio.

---

## Visão geral da arquitetura

```
GitHub Pages (front-end)          Firebase (back-end gratuito)
┌──────────────────────┐          ┌─────────────────────────┐
│  index.html          │◄────────►│  Firestore Database     │
│  admin.html          │          │  Authentication         │
└──────────────────────┘          └─────────────────────────┘
```

O **Firestore** guarda dois tipos de dados:

| Coleção       | O que guarda                        | Quem acessa           |
|---------------|-------------------------------------|-----------------------|
| `bookings`    | Todos os dados do agendamento       | Apenas admin (auth)   |
| `taken`       | Quais horários estão ocupados por data | Site público (leitura) |

---

## Passo 1 — Criar o projeto Firebase

1. Acesse [console.firebase.google.com](https://console.firebase.google.com) com a conta Google da Patricia
2. Clique em **Adicionar projeto**
3. Nome do projeto: `patricia-moura-lash` (ou qualquer nome)
4. Desative o Google Analytics (não é necessário) → **Criar projeto**
5. Aguarde a criação (cerca de 30 segundos)

---

## Passo 2 — Criar o banco de dados (Firestore)

1. No menu lateral, clique em **Firestore Database**
2. Clique em **Criar banco de dados**
3. Escolha **Iniciar no modo de produção** → **Avançar**
4. Selecione a localização: **southamerica-east1 (São Paulo)** → **Ativar**
5. Aguarde o banco ser criado

### Configurar as regras de segurança

1. Na página do Firestore, clique na aba **Regras**
2. Substitua todo o conteúdo pelas regras abaixo:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Horários ocupados — leitura pública, escrita pública (para novos agendamentos)
    match /taken/{date} {
      allow read: if true;
      allow write: if true;
    }

    // Agendamentos completos — qualquer um cria, só admin lê/edita/exclui
    match /bookings/{id} {
      allow create: if true;
      allow read, update, delete: if request.auth != null;
    }

    // Dias bloqueados — leitura pública (site verifica), escrita só admin
    match /blockedDays/{date} {
      allow read: if true;
      allow write: if request.auth != null;
    }
  }
}
```

3. Clique em **Publicar**

---

## Passo 3 — Configurar a autenticação (login do admin)

1. No menu lateral, clique em **Authentication**
2. Clique em **Primeiros passos**
3. Na aba **Método de login**, clique em **E-mail/senha**
4. Ative a opção **E-mail/senha** → **Salvar**

### Criar a conta da Patricia (admin)

1. Na aba **Usuários**, clique em **Adicionar usuário**
2. Preencha:
   - **E-mail:** o e-mail da Patricia (ex: `patricia@gmail.com`)
   - **Senha:** uma senha forte (mínimo 8 caracteres)
3. Clique em **Adicionar usuário**

> Guarde bem o e-mail e senha — serão usados para entrar no painel admin.

### Adicionar o domínio do GitHub Pages (obrigatório)

1. Ainda em **Authentication**, clique na aba **Configurações**
2. Em **Domínios autorizados**, clique em **Adicionar domínio**
3. Digite o domínio do GitHub Pages: `seuusuario.github.io`
   - Substitua `seuusuario` pelo nome de usuário do GitHub
4. Clique em **Adicionar**

---

## Passo 4 — Obter as configurações do projeto

1. Clique na engrenagem ⚙️ ao lado de **Visão geral do projeto** → **Configurações do projeto**
2. Role a página até a seção **Seus apps**
3. Clique no ícone **`</>`** (Web) para adicionar um app web
4. Nome do app: `patricia-moura-site` → clique em **Registrar app**
5. Você verá um bloco de código parecido com este:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSyXXXXXXXXXXXXXXXXXXXXX",
  authDomain: "patricia-moura-lash.firebaseapp.com",
  projectId: "patricia-moura-lash",
  storageBucket: "patricia-moura-lash.appspot.com",
  messagingSenderId: "123456789012",
  appId: "1:123456789012:web:abc123def456"
};
```

6. **Copie estas informações** — você vai precisar delas no próximo passo
7. Clique em **Continuar no console**

---

## Passo 5 — Colar a configuração nos arquivos HTML

Você precisa colar as mesmas configurações em **dois arquivos**:

### No arquivo `index.html`

Abra o arquivo e encontre (pressione `Ctrl+F` e busque por `COLE_AQUI`):

```javascript
const firebaseConfig = {
  apiKey:            'COLE_AQUI',
  authDomain:        'COLE_AQUI',
  projectId:         'COLE_AQUI',
  storageBucket:     'COLE_AQUI',
  messagingSenderId: 'COLE_AQUI',
  appId:             'COLE_AQUI'
};
```

Substitua cada `'COLE_AQUI'` pelo valor correspondente do Firebase. Exemplo:

```javascript
const firebaseConfig = {
  apiKey:            'AIzaSyXXXXXXXXXXXXXXXXXXXXX',
  authDomain:        'patricia-moura-lash.firebaseapp.com',
  projectId:         'patricia-moura-lash',
  storageBucket:     'patricia-moura-lash.appspot.com',
  messagingSenderId: '123456789012',
  appId:             '1:123456789012:web:abc123def456'
};
```

### No arquivo `admin.html`

Encontre o mesmo bloco `firebaseConfig` e substitua da mesma forma.

> As configurações são **idênticas** nos dois arquivos.

---

## Passo 6 — Publicar no GitHub Pages

1. Faça commit e push de todos os arquivos para o GitHub:
   ```
   git add .
   git commit -m "Integra Firebase"
   git push
   ```
2. No GitHub, acesse o repositório → **Settings** → **Pages**
3. Em **Source**, selecione **Deploy from a branch** → branch `main` → pasta `/ (root)`
4. Clique em **Save**
5. Aguarde 1-2 minutos. O site estará disponível em:
   `https://seuusuario.github.io/nome-do-repositorio/`

---

## Como acessar o painel admin

1. Abra `https://seuusuario.github.io/nome-do-repositorio/admin.html`
2. Digite o **e-mail** e **senha** criados no Passo 3
3. O painel carrega todos os agendamentos do Firestore

---

## Estrutura dos dados no Firestore

### Coleção `bookings` (um documento por agendamento)
```
bookings/
  {id-automatico}/
    createdAt:  Timestamp
    name:       "Ana Paula Ferreira"
    date:       "2026-05-15"
    time:       "09:00"
    service:    "Volume Egípcio"
    price:      185
    phone:      "(34) 99999-1234"
    email:      "ana@email.com"
    notes:      "Alergia a látex"
    status:     "confirmado"
```

### Coleção `taken` (um documento por data com horários ocupados)
```
taken/
  "2026-05-15"/
    "09:00": "id-do-booking"
    "14:00": "id-do-booking"
```

---

## Limites do plano gratuito (Spark)

| Recurso            | Limite gratuito        | Uso esperado      |
|--------------------|------------------------|-------------------|
| Leituras Firestore | 50.000/dia             | ~200 visitas/dia  |
| Escritas Firestore | 20.000/dia             | ~50 agendamentos  |
| Armazenamento      | 1 GB                   | Anos de dados     |
| Autenticação       | Ilimitado              | —                 |

Para um estúdio pequeno, o plano gratuito é mais do que suficiente.

---

## Dúvidas frequentes

**Os horários não aparecem bloqueados.**
Verifique se o `firebaseConfig` foi colado corretamente no `index.html`, sem espaços extras e com as aspas simples.

**O painel admin mostra "Firebase não configurado".**
Verifique se o `firebaseConfig` no `admin.html` também está preenchido.

**Erro de autenticação no painel.**
Confirme que o domínio do GitHub Pages foi adicionado nos domínios autorizados do Firebase Authentication (Passo 3).

**Quero trocar a senha do admin.**
Acesse Firebase Console → Authentication → Usuários → clique nos três pontos ao lado do e-mail → **Redefinir senha**.

**Quero adicionar outro e-mail admin.**
Firebase Console → Authentication → Adicionar usuário. Qualquer usuário autenticado terá acesso ao painel admin.
