# Dôa — plataforma de doação de alimentos

Conecta **doadores** (mercados, restaurantes, pessoas) a **instituições** que recebem alimentos, evitando desperdício.

Projeto acadêmico (BRADEMO — Mobile, BRADWBK — Back-end, Projeto de TIC).
Autores: Áquila Matheus de Oliveira (BP3054306) e Eduardo Schoof (BP3060357).

## Estrutura
| Pasta | Conteúdo |
|---|---|
| `backend/` | API REST em Spring Boot 3.5 / Java 21 (JWT, JPA, WebSocket, Swagger, FCM) |
| `app/` | Aplicativo Flutter/Dart (Provider, go_router, camada de rede, push, câmera/galeria) |
| `banco/` | Scripts MySQL (`criar-usuario.sql`, `schema.sql`) |
| `backend/postman/` | Coleção Postman (61 requisições) |
| `backend/docs/` | `API.md` (especificação) e `MODELO.md` (DER) |

## Funcionalidades
Cadastro/login com perfis (doador, instituição, admin) · CRUD de doações com até 5 fotos · categorias · busca por categoria e raio · solicitações/reserva · estoque · chat em tempo real (WebSocket) · notificações push (FCM) · impacto do doador · aprovação de instituições pelo admin.

## Modelo de dados
User, Donor, Institution, FoodCategory, Donation, DonationPhoto, StockItem, Message.
Relações: Doador 1:N Doação; Instituição N:N Categoria.

## Como rodar (resumo)
1. **Banco:** MySQL (`banco/criar-usuario.sql`) ou, sem configurar nada, H2 em memória.
2. **API:** `cd backend && mvn spring-boot:run` (perfil MySQL: `SPRING_PROFILES_ACTIVE=mysql`) → http://localhost:8080 · Swagger em `/swagger-ui.html`.
3. **App:** `cd app && flutter pub get && flutter run -d chrome` (emulador Android: `--dart-define=API_URL=http://10.0.2.2:8080`).

Passo a passo completo em [COMO-INICIAR.md](COMO-INICIAR.md).

## Contas de demonstração
admin@doa.com / admin123 · doador@doa.com / doador123 · mercado@doa.com / mercado123 · ong@doa.com / ong123

## Documentação
- API: `backend/docs/API.md` e coleção `backend/postman/`
- Protótipo Figma: https://www.figma.com/design/l6tBYDLJyGlIXEHxyTjQeZ

## Segurança
Chaves de serviço do Firebase (`FCM_CREDENTIALS_PATH`) ficam **fora** do repositório. O `google-services.json` do app contém apenas configuração pública do cliente.
