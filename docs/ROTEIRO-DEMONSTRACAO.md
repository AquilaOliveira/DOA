# Dôa — roteiro de testes e demonstração

Ordem da demonstração: **1) banco → 2) API → 3) Postman → 4) WebSocket/Swagger → 5) app no emulador**.
Tempo sugerido para o vídeo (máx. 15 min): banco 2 min · API 1 min · Postman 6 min · WebSocket 2 min · app 3 min · fechamento 1 min.

## 0. Antes de gravar
- MySQL 8 rodando (ou `docker compose up --build` na pasta `backend`).
- Postman instalado e a coleção `backend/postman/Doa-API.postman_collection.json` importada (Import → arquivo).
- Emulador Android aberto (Medium Phone) com a opção *Location* configurada (ver passo 5).
- Para limpar os dados e começar do zero: no MySQL, `DROP DATABASE doa;` e rode `banco/criar-usuario.sql` de novo.

## 1. Mostrar o banco (MySQL Workbench)
1. Conecte em `localhost:3306` com o usuário `doa` (senha `doa`).
2. Rode `banco/criar-usuario.sql` (cria o schema `doa` e o usuário) e mostre o schema vazio. Se a API já tiver rodado, mostre as tabelas.
3. Fale do modelo: `users`, `donors`, `institutions`, `food_categories`, `donations`, `donation_photos`, `stock_items`, `messages` e a tabela de junção `institution_category` (N:N).
4. Mostre as relações: Reverse Engineer (Database → Reverse Engineer) ou o DER do documento.
5. Deixe uma aba pronta com consultas para usar depois do Postman:
```sql
USE doa;
SELECT id, email, role, active FROM users;
SELECT id, name, city, latitude, longitude FROM donors;
SELECT id, name, status FROM institutions;
SELECT id, description, status, reserved_by_id FROM donations;
SELECT id, institution_id, description, quantity FROM stock_items;
SELECT id, donation_id, content FROM messages;
```

## 2. Subir a API
1. No IntelliJ/VS Code, defina as variáveis: `SPRING_PROFILES_ACTIVE=mysql`, `DB_HOST=localhost`, `DB_USER=doa`, `DB_PASSWORD=doa`.
   (Opcional, só para push: `FCM_CREDENTIALS_PATH=<caminho do JSON do Firebase>`.)
2. Execute a classe principal (`DoaApiApplication`) ou `mvn spring-boot:run`.
3. Aguarde `Started DoaApiApplication`. No Workbench, atualize (Ctrl+R) e mostre que as tabelas foram criadas e o `users` já tem as contas de demonstração.
4. Abra http://localhost:8080/swagger-ui.html para mostrar a documentação dos endpoints.

## 3. Testes no Postman
A coleção tem 8 pastas e 61 requisições, cada uma com testes automáticos (`pm.test`). Os tokens JWT são salvos sozinhos em variáveis da coleção; a variável `baseUrl` já aponta para `http://localhost:8080`.

### 3.1 Rodar tudo de uma vez (resultado para o PDF)
1. Clique na coleção → **Run** (Runner).
2. Deixe todas as pastas marcadas, ordem original, 1 iteração, **Persist responses** desmarcado.
3. Clique em **Run Doa API**. Aguarde terminar.
4. Tire print do resumo (Passed/Failed). Esse é o principal comprovante do PDF de especificação.
5. Atenção: as requisições **“Foto: enviar (201)”** e a de arquivo inválido usam `form-data` com arquivo. Antes de rodar, abra cada uma, na aba *Body* selecione o campo `file` e escolha uma imagem JPG/PNG no seu computador (e um `.txt` na de “arquivo que não é imagem”). Sem isso elas falham.
6. O Runner só funciona bem uma vez por banco: algumas requisições criam dados fixos (ex.: cadastro público). Para repetir, reinicie a API com o banco zerado (passo 0) ou use outro e-mail.

### 3.2 Rodar pasta por pasta (para explicar no vídeo)
Sempre começar pela pasta **1. Autenticação** (salva `adminToken`, `donorToken`, `institutionToken`).

| Pasta | O que mostrar | Resultado esperado |
|---|---|---|
| 1. Autenticação | Login dos 3 perfis; senha errada; cadastro público de doador; tentativa de criar ADMIN; rota sem token | 200 · 401 · 201 · 403 · 401 |
| 2. Usuários (admin) | CRUD de usuário; registrar token FCM; doador listando usuários | 200/201/204 · 403 para doador |
| 3. Categorias (admin) | CRUD completo; excluir categoria em uso; doador criando categoria | 201/200/204 · **409** · 403 |
| 4. Doadores | Meu cadastro, atualizar, duplicado, listar (admin) | 200 · 409 |
| 5. Instituições + categorias aceitas | Relação **N:N**; pendentes; aprovar; aprovar de novo; rejeitar sem justificativa | 200 · **409** · **400** |
| 6. Doações | Criar, fotos, buscar por distância, solicitar, chat, confirmar recebimento, cancelar entregue, excluir | 201/200/204 · 400 (vencida/arquivo inválido) · 409 (editar reservada / cancelar entregue) |
| 7. Estoque | Listar, adicionar, atualizar, descartar | 200/201/204 |
| 8. Impacto | Doador, instituição e consolidado | 200 |

Em cada requisição, mostre: o método e a URL, o corpo enviado, o status devolvido e a aba **Test Results** (verde). Entre as pastas, volte ao Workbench e rode a consulta correspondente para provar que o dado foi gravado (ex.: após “Criar doação”, `SELECT * FROM donations;`).

### 3.3 Pontos que atendem os critérios da disciplina
- **4 CRUDs completos:** usuários, categorias, doações e estoque (pastas 2, 3, 6 e 7). Doadores e instituições também têm cadastro e edição.
- **Relacionamento 1:N:** doador → doações; doação → fotos; instituição → estoque.
- **Relacionamento N:N:** instituição ↔ categorias aceitas (pasta 5).
- **Segurança:** 401 sem token, 403 por perfil, bloqueio após 3 logins errados (faça 3 logins errados com um usuário de teste e mostre o 4º bloqueado; o desbloqueio automático leva 30 minutos).
- **Regras de negócio:** doação vencida (400), editar doação reservada (409), limite de 5 reservas ativas, reserva que expira em 24 h (rotina agendada).

## 4. WebSocket (chat em tempo real) e Swagger
1. Abra **duas abas** em http://localhost:8080/ws-demo/index.html.
2. Na aba A, entre como `doador@doa.com` / `doador123`; na aba B, como `ong@doa.com` / `ong123`.
3. Informe o ID da doação reservada (crie e reserve uma pelo Postman, pasta 6, se necessário) e conecte as duas.
4. Envie mensagens de um lado e mostre que chegam no outro na hora. Depois mostre a mensagem gravada: `SELECT * FROM messages;`.
5. Swagger (http://localhost:8080/swagger-ui.html): clique em **Authorize**, cole o token e execute um GET para mostrar que a documentação também é executável.

## 5. App (emulador Android)
1. Com a API rodando: na pasta `app`, `flutter pub get` e
   `flutter run --dart-define=API_URL=http://10.0.2.2:8080`
   (a pasta do projeto não pode ter acento no caminho).
2. Localização do emulador: botão “⋯” da barra lateral (Extended controls) → **Location** → escolha um ponto → **Set Location**. Sem isso o GPS não retorna nada.
3. Roteiro:
   - Cadastre um doador novo (aceite a permissão de localização e toque em “Usar minha localização”). Depois mostre no Workbench que `donors` ganhou `latitude`/`longitude`.
   - **Meu cadastro** (ícone no topo): edite nome/telefone e salve; mostre a alteração no banco.
   - Doador → **Nova doação** com foto (câmera/galeria).
   - Instituição (`ong@doa.com`) → **Buscar doações** (ordenadas por distância) → **Solicitar**. Mostre a notificação push (precisa do Firebase configurado).
   - Chat da doação nos dois perfis.
   - Instituição confirma recebimento → item aparece em **Estoque**.
   - Admin (`admin@doa.com`) → aprovar `ong2@doa.com` e gerenciar categorias.
4. Fechamento: mostre **Impacto** no app e, no Workbench, o `SELECT` final de `donations`/`stock_items`.

## 6. Evidências para o PDF de especificação (BRADWBK)
Guarde prints de: (a) Runner com todos os testes passando; (b) uma requisição por pasta com status e corpo; (c) o Workbench com as tabelas e dados gravados; (d) a página do WebSocket com mensagens trocadas; (e) o Swagger. Nomeie os arquivos na ordem dos passos acima.

## 7. Se algo der errado
| Sintoma | O que fazer |
|---|---|
| Postman 401 em tudo | Rode a pasta 1 de novo (tokens expiraram) |
| 409 no cadastro de doador | E-mail já usado; troque o e-mail ou zere o banco |
| API não sobe no MySQL | Confira as variáveis `DB_*` e se o MySQL está na porta 3306 |
| Emulador sem localização | Extended controls → Location → Set Location |
| Emulador não conecta na API | Use `10.0.2.2:8080`, nunca `localhost` |
| Não chega push | Emulador com Google Play, permissão de notificação aceita e `FCM_CREDENTIALS_PATH` configurado |
