# Dôa — como iniciar tudo

Estrutura: `backend/` (API Spring Boot), `app/` (Flutter), `banco/` (scripts MySQL).
Ordem: banco → API → app.

## 0. Pré-requisitos
- Java 21 e Maven (ou o Maven embutido no IntelliJ/VS Code)
- Flutter SDK (e Android Studio com um emulador "Google Play" para o push)
- MySQL 8 **ou** Docker Desktop

## 1. Banco de dados (MySQL) — escolha UMA opção

**Opção A — Docker (mais fácil: sobe banco e API juntos)**
```
cd backend
docker compose up --build
```
Pronto: MySQL em localhost:3306 e API em http://localhost:8080. Pule para o passo 3.

**Opção B — MySQL instalado na máquina**
1. Abra o MySQL (Workbench ou terminal) como root e rode `banco/criar-usuario.sql`
   (cria o banco `doa` e o usuário `doa` / senha `doa`).
2. As tabelas são criadas sozinhas pela API na primeira execução.
   (O arquivo `banco/schema.sql` é a versão manual, só se quiser criá-las à mão; nesse caso use `DDL_AUTO=validate`.)

**Opção C — sem MySQL (só para testar rápido)**
Não configure nada: a API usa um banco H2 em memória, que zera a cada reinício.

## 2. API (backend) — Opções B e C
```
cd backend
mvn test  # opcional: roda os testes
```
Com MySQL (PowerShell):
```
$env:SPRING_PROFILES_ACTIVE="mysql"
$env:DB_HOST="localhost"; $env:DB_USER="doa"; $env:DB_PASSWORD="doa"
$env:FCM_CREDENTIALS_PATH="C:\caminho\fora-do-repo\firebase-chave.json"   # só para push
mvn spring-boot:run
```
Sem MySQL: apenas `mvn spring-boot:run`.

Conferir: http://localhost:8080/swagger-ui.html abre a documentação.
Demo do WebSocket: http://localhost:8080/ws-demo/index.html

Contas de demonstração (criadas automaticamente):
| Perfil | E-mail | Senha |
|---|---|---|
| Admin | admin@doa.com | admin123 |
| Doador | doador@doa.com | doador123 |
| Doador | mercado@doa.com | mercado123 |
| Instituição (aprovada) | ong@doa.com | ong123 |
| Instituição (pendente) | ong2@doa.com | ong123 |

## 3. App (Flutter)
```
cd app
flutter pub get
flutter analyze               # opcional
flutter test                  # opcional
```
- **Chrome:** `flutter run -d chrome` (usa http://localhost:8080)
- **Emulador Android:** `flutter run --dart-define=API_URL=http://10.0.2.2:8080`
- **Celular físico (mesma rede Wi-Fi):** `flutter run --dart-define=API_URL=http://IP_DO_PC:8080`
  (descubra o IP com `ipconfig`; libere a porta 8080 no firewall)

Push (FCM) só funciona no Android, com a chave do Firebase configurada na API (passo 2).

## 4. Roteiro rápido de teste
1. Entre como `doador@doa.com` → Nova doação (com foto da câmera/galeria).
2. Entre como `ong@doa.com` → Buscar doações → Solicitar. (Com o app em segundo plano, deve chegar o push.)
3. Abra o chat da doação reservada nos dois lados.
4. Instituição confirma o recebimento → o item aparece no Estoque.
5. Entre como `admin@doa.com` → Instituições (aprovar `ong2@doa.com`) e Categorias.

## 5. Problemas comuns
- **"Não foi possível conectar ao servidor":** API fora do ar ou URL errada (emulador usa 10.0.2.2, não localhost).
- **API não conecta no MySQL:** confira `DB_USER`/`DB_PASSWORD` e se o MySQL está na porta 3306.
- **Não chega push:** use emulador com Google Play, aceite a permissão de notificação e confira `FCM_CREDENTIALS_PATH`.
- **Segredos:** nunca envie ao Git o JSON de conta de serviço do Firebase. Em produção troque `JWT_SECRET` e a senha do admin (`ADMIN_PASSWORD`).
