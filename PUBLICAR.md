# PUBLICAR — Gestor de Projetos STEAM (Quadros Kanban)

Guia de publicação do `gestor-projetos/index.html`.

> O app já vem configurado para o projeto Firebase **`plan-sc-ecb9d`** (o mesmo
> do Planejador BNCC), mas o **Firestore ainda precisa ser ativado** e as
> **regras de segurança configuradas** para o app funcionar.

---

## 1. Ativar Authentication (E-mail/senha)

1. Acesse https://console.firebase.google.com
2. Abra o projeto **`plan-sc-ecb9d`**.
3. Menu **Build → Authentication → Sign-in method** (métodos de login).
4. Ative **E-mail/senha** e clique em **Salvar**.

## 2. Ativar o Firestore Database

1. Ainda no console, menu **Build → Firestore Database**.
2. Clique em **Criar banco de dados**.
3. Em **Modo de produção** (recomendado).
4. Localização sugerida: **southamerica-east1** (São Paulo) — não é possível
   mudar depois.
5. Clique em **Criar**.

## 3. Configurar as regras do Firestore

1. Abra **Firestore Database → Regras**.
2. Apague o conteúdo atual e cole as regras abaixo.
3. Clique em **Publicar**.

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Professor é o usuário cujo perfil em /usuarios tem perfil == 'professor'.
    function professor() {
      return request.auth != null &&
        get(/databases/$(database)/documents/usuarios/$(request.auth.uid)).data.perfil == 'professor';
    }

    // Perfis: cada usuário cria e edita somente o próprio perfil; todos podem ler.
    match /usuarios/{uid} {
      allow read: if request.auth != null;
      allow create: if request.auth != null && request.auth.uid == uid;
      allow update: if request.auth != null && request.auth.uid == uid;
    }

    // Projetos: leitura para qualquer usuário logado; gravação só professor.
    match /projetos/{pid} {
      allow read: if request.auth != null;
      allow create, update, delete: if professor();
    }

    // Grupos de trabalho: leitura para qualquer usuário logado; gravação só professor.
    match /grupos/{gid} {
      allow read: if request.auth != null;
      allow create, update, delete: if professor();
    }

    // Cartões: leitura para qualquer usuário logado; gravação só professor.
    match /cartoes/{cid} {
      allow read: if request.auth != null;
      allow create, update, delete: if professor();
    }
  }
}
```

### Opcional: permitir que alunos editem/prendem seus próprios cartões

Se quiser manter as permissões 100% no app (aluno pode marcar checklist,
comentar e mover o próprio grupo), troque **apenas** o bloco de `/cartoes` por:

```js
    match /cartoes/{cid} {
      allow read, get: if request.auth != null;
      allow create, update, delete: if professor();
    }
```

> 📌 Na versão 1, a edição por alunos já funciona visualmente no navegador
> (o app exige estar no grupo do cartão), mas o **Firestore bloqueia** gravações
> de cartões feitas por alunos nas regras acima. Se quiser liberar de verdade,
> use a regra opcional. As regras são o "portão final" de segurança.

## 4. Testar localmente (opcional)

Abra o `index.html` direto no navegador (duplo clique). A autenticação e o
Firestore funcionam em `file://`. É o jeito mais rápido de testar.

## 5. Publicar na web

### Opção A — GitHub Pages (grátis, recomendado)

1. Crie um repositório (ex.: `gestor-projetos`) e envie o `index.html` para a
   branch principal (ou para uma pasta `docs/`).
2. No GitHub: **Settings → Pages → Source → Deploy from a branch** → escolha a
   branch e, se usou a pasta `docs/`, selecione `/docs`.
3. Acesse `https://SEU-USUARIO.github.io/gestor-projetos/`.

### Opção B — Firebase Hosting (grátis)

```bash
npm install -g firebase-tools
firebase login
firebase init hosting
# Pasta pública: .  | Aplicação de página única: Sim
firebase deploy
```

## 6. Primeiros passos no app

1. Abra o app e **crie a conta do professor** (perfil: **Professor**).
2. Crie um **projeto** no botão "＋ Novo projeto" (as 5 etapas STEAM já vêm
   prontas e podem ser reescritas).
3. Na tela do quadro, clique nos chips de grupo (ou no botão **Grupos** nas
   abas do topo) para **criar grupos e marcar os alunos** como membros.
4. Crie cartões em cada etapa, com prazos, prioridade, responsáveis,
   etiquetas, checklist e comentários.
5. Os **alunos** criam suas contas (perfil: **Aluno**) — assim que forem
   adicionados a um grupo pelo professor, passam a ver apenas o quadro do(s)
   projeto(s) do próprio grupo.

## Observações

- O `index.html` usa somente o SDK Firebase compat (gratuito). Dados ficam em
  nuvem e são sincronizados em tempo real entre professor e alunos.
- Para restaurar um **cartão/projeto arquivado**, basta desmarcar
  `arquivado: true` no Firestore (na versão 1 o arquivado não tem tela de
  restauração).
- Os e-mails de teste usado no login precisam ser confirmados? **Não** — a
  versão atual aceita login sem confirmação de e-mail, mas o app envia um
  e-mail de boas-vindas/verificação ao criar a conta (opcional).