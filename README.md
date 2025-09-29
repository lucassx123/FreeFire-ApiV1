# Free Fire Apis

Documentação dos endpoints HTTP públicos, com exemplos reais de requisição e resposta em **JSON**.

**Base URL**
```
https://freefirefwx-beta.squareweb.app
```

> **Formato das respostas**
>
> - Sucesso normalmente retorna `{ "success": true, ... }` (ou `"status": "success"` em alguns casos).
> - Erros retornam `{ "success": false, "error": "<código>" }` (ou `"status": "error"`), com **HTTP status** apropriado (400, 401, 404, 500…).
> - Campos seguem os nomes originais dos *protos* (quando houver), preservados.

## Regiões suportadas

- **Padrão do repositório** (`covers_regions`): `br`, `sac`, `us`, `na`, `latam`.
- **Leaderboard / Pass**: atualmente **`br`** e **`ind`**.
- Você pode ampliar regiões via `accounts.json` (os códigos devem constar em `covers_regions`).

---

## 1) Buscar jogador por **nickname**
**GET** `https://freefirefwx-beta.squareweb.app/api/search_by_nickname?nickname=<string>&region=<br|...|all>`

- **Parâmetros**
  - `nickname` *(string, obrigatório)*
  - `region` *(string, obrigatório)* — `br`, `sac`, `us`, `na`, `latam` ou `all` (pesquisa paralela em todas as regiões configuradas)

**Exemplo**
```
GET https://freefirefwx-beta.squareweb.app/api/search_by_nickname?nickname=Biel&region=br
```

**Resposta (200)**
```json
{
  "success": true,
  "count": 2,
  "accountBasicInfo": [
    {
      "accountId": "12345678",
      "nickname": "Biel",
      "region": "BR",
      "level": 65,
      "headPic": "900000013",
      "bannerId": "900000014"
    },
    {
      "accountId": "87654321",
      "nickname": "Bielzin",
      "region": "BR",
      "level": 51,
      "headPic": "900000055",
      "bannerId": "900000099"
    }
  ]
}
```

**Erros**
- 400 `{"success": false, "error": "invalid_request"}`
- 500 `{"success": false, "error": "search_failed"}`

---

## 2) Informações do jogador (por **UID**) + imagens opcionais
**GET** `https://freefirefwx-beta.squareweb.app/api/info_player?uid=<id>&region=<br|...>&clothes=<true|false>`

- **Parâmetros**
  - `uid` *(string numérica, obrigatório)*
  - `region` *(string, obrigatório)*
  - `clothes` *(boolean string, opcional; padrão: `false`)* — se `true`, gera preview de **roupas**.

**Exemplo**
```
GET https://freefirefwx-beta.squareweb.app/api/info_player?uid=12345678&region=br&clothes=true
```

**Resposta (200)**
```json
{
  "basicInfo": {
    "accountId": "12345678",
    "nickname": "JogadorX",
    "region": "BR",
    "level": 60,
    "headPic": "900000013",
    "bannerId": "900000014",
    "avatars": {
      "png": "https://freefirefwx-beta.squareweb.app/api/v1/ff/avatar/png?image=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/avatar-12345678.png",
      "webp": "https://freefirefwx-beta.squareweb.app/api/v1/ff/avatar/webp?image=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/avatar-12345678.webp"
    }
  },
  "profileInfo": {
    "clothes": {
      "ids": [ /* ids de roupas */ ],
      "images": [
        "https://freefirefwx-beta.squareweb.app/api/v1/ff/clothes/jpeg?image=YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY/outfit-12345678.jpeg",
        "https://freefirefwx-beta.squareweb.app/api/v1/ff/clothes/webp?image=YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY/outfit-12345678.webp"
      ]
    }
  },
  "clanBasicInfo": {
    "clanName": "MinhaGuild",
    "clanId": "55555"
  }
}
```

**Erros**
- 400 `{"success": false, "error": "invalid_request"}`
- 404 `{"error": "player_not_found"}`
- 500 `{"error": "system_error"}`

> **Imagens locais**
> - Endpoint de imagens: `GET https://freefirefwx-beta.squareweb.app/api/v1/ff/<avatar|clothes>/<png|webp|jpeg>?image=<hash_dir>/<arquivo>`
> - Retorna o arquivo da pasta `AVATARS/` ou `CLOTHES/` (404 se não existir).

---

## 3) Buscar **Guild** por ID
**GET** `https://freefirefwx-beta.squareweb.app/api/search_guild_id?id=<guild_id>&region=<br|...>`

- **Parâmetros**: `id` *(string numérica, obrigatório)*, `region` *(string, obrigatório)*

**Resposta (200)**
```json
{
  "success": true,
  "clanInfo": {
    "clanId": "12345",
    "clanName": "TOPGuild",
    "region": "BR",
    "level": 5,
    "membersCount": 49,
    "description": "..."
  }
}
```

**Erros**: 400 `invalid_request|invalid_guild_id`, 404 `guild_not_found`.

---

## 4) Descobrir **região** por UID
**GET** `https://freefirefwx-beta.squareweb.app/api/region?id=<uid>`

**Resposta (200)**
```json
{
  "nickname": "JogadorX",
  "region": "BR",
  "account_uid": "12345678",
  "openid_auth": "xxxxxxxxxxxxxxxx"
}
```

**Erros**: 400 `missing uid|invalid_uid`, 408 `timeout`.

---

## 5) Alterar **nickname**
**GET** `https://freefirefwx-beta.squareweb.app/api/change_nick?nickname=<novo>&access_token=<game_token>`

**Respostas**
- **200**
```json
{
  "success": true,
  "message": "Nickname changed successfully",
  "old_nickname": "OldName",
  "new_nickname": "NewName",
  "account_id": "12345678"
}
```
- **400** `AccessToken invalid | Invalid_Name_Length | this nickname is already in use | the account does not have a nickname change`
- **500** `Failed to change nickname | Failed to parse response`

---

## 6) **Wishlist** (itens do jogador)
### 6.1 Listar
**GET** `https://freefirefwx-beta.squareweb.app/api/wishlist?id=<uid>&region=<br|...>`

**Resposta (200)**
```json
{
  "success": true,
  "wishlistBasicInfo": {
    "Items": [
      { "item_id": 123, "quantity": 1 },
      { "item_id": 456, "quantity": 2 }
    ]
  }
}
```
**Erros**: 400 `invalid_request|invalid_id|region_not_implemented`, 404 `no items found in this player's wishlist`.

### 6.2 Adicionar item
**GET/POST** `https://freefirefwx-beta.squareweb.app/api/wishlist_add?item_id=<id>&region=<br|...>&(access_token|jwt|uid&password)`

**Resposta (200)** `{ "status": "success", "message": "Items added to wishlist." }`  
**Erros**: 400 parâmetros ausentes; 401 `INVALID_TOKEN`; 500 `FAILED_TO_ADD`.

### 6.3 Remover item
**GET/POST** `https://freefirefwx-beta.squareweb.app/api/wishlist_remove?item_id=<id>&region=<br|...>&(access_token|jwt|uid&password)`

**Resposta (200)** `{ "status": "success", "message": "Items removed from wishlist." }`  
**Erros**: 400 parâmetros ausentes; 401 `INVALID_TOKEN`; 500 `FAILED_TO_REMOVE`.

---

## 7) Alterar **bio**
**GET** `https://freefirefwx-beta.squareweb.app/api/bio_change?new_bio=<string>&access_token=<game_token>&region=<br|...>`

**Resposta (200)**
```json
{
  "success": "true",
  "message": "Bio Updated Successfully!",
  "new_bio": "Olá mundo!",
  "old_bio": "Minha bio antiga",
  "nickname": "JogadorX"
}
```
**Erros**: 400 `invalid request|AccessToken invalid|region_not_implemented`, 500 `Failed to update bio`.

---

## 8) **Login**
**GET** `https://freefirefwx-beta.squareweb.app/api/login?access_token=<game_token>`
<br>ou
**GET** `https://freefirefwx-beta.squareweb.app/api/login?uid=<guest_uid>&password=<guest_password>&emulator=<true|false>`

**Resposta (200)**
```json
{
  "success": true,
  "jwt": "<bearer jwt>",
  "emulator": false,
  "FF_version": "1.111.11",
  "FF_client": "1.111.1",
  "auth": "OB49"
}
```
**Erros**: 400 `missing access_token or uid/password | INVALID_TOKEN`, 401 `INVALID_GUEST`.

---

## 9) **Guest** – obter `access_token`
**GET** `https://freefirefwx-beta.squareweb.app/api/guest?guest_uid=<uid>&guest_password=<pwd>`

**Resposta (200)**
```json
{
  "success": true,
  "message": "guest account information successfully obtained!",
  "guestBasicInfo": {
    "access_token": "<token>",
    "expires_in": 86400,
    "open_id": "...",
    "platform": "..."
  }
}
```
**Erros**: 400 `guest password or uid is incorrect`, 500 `failed to connect to api`.

---

## 10) **Inspecionar token**
**GET** `https://freefirefwx-beta.squareweb.app/api/v1/token?access_token=<token>&token_type=<game|web>`

- **Se `token_type=web`** (ou `access_token` muito longo):
  **Resposta (200)** (exemplo resumido)
  ```json
  {
    "success": true,
    "message": "Token valid",
    "player_nickname": "JogadorX",
    "player_id": "12345678",
    "player_region": "BR",
    "player_level": 60,
    "player_exp": 123456
  }
  ```
- **Se `token_type=game`**:
  ```json
  {
    "success": true,
    "message": "token information obtained successfully!",
    "tokenBasicInfo": {
      "uid": "12345678",
      "open_id": "...",
      "platform": "..."
    }
  }
  ```
**Erros**: 400 `AccessToken Invalid`, 500 `failed to inspect token`.

---

## 11) **Leaderboard** (pontos)
**GET** `https://freefirefwx-beta.squareweb.app/api/leaderboard?region=<br|ind>`

**Resposta (200)**
```json
{
  "success": "true",
  "message": "leaderboard information successfully obtained",
  "leaderboardBasicInfo": [
    { "playerId": "1", "playerNickname": "AAA", "playerPosition": 1, "playerPoints": 12345 },
    { "playerId": "2", "playerNickname": "BBB", "playerPosition": 2, "playerPoints": 12001 }
  ]
}
```
**Erros**: 400 `region_not_implemented`, 404 `no players found`, 500 `no token available for this region|internal server error`.

---

## 12) **Leaderboard Pass** (regional)
**GET** `https://freefirefwx-beta.squareweb.app/api/leaderboard_pass?region=<br|ind>`

**Resposta (200)**
```json
{
  "success": "true",
  "message": "pass leaderboard information successfully obtained",
  "leaderboardPass": [
    { "nickname": "AAA", "region": "BR", "passLevel": 150, "playerPosition": 1 }
  ]
}
```

---

## 13) **Leaderboard Pass Global**
**GET** `https://freefirefwx-beta.squareweb.app/api/leaderboard_pass_global`

**Resposta (200)**
```json
{
  "success": "true",
  "message": "global pass leaderboard information successfully obtained",
  "leaderboardPass": [
    { "nickname": "AAA", "region": "BR", "passLevel": 150, "playerPosition": 1 }
  ]
}
```

---

## 14) **Craftland** (mapa por código)
**GET** `https://freefirefwx-beta.squareweb.app/api/craftland?map_code=<string>&region=<br|...>`

**Resposta (200)**
```json
{
  "success": true,
  "message": "workshop info obtained",
  "workshopInfo": {
    "authorId": "999999",
    "authorName": "PlayerZ",
    "workshopDescription": "Mapa insano",
    "updateTime": 1722890442,
    "peoplePlayingNow": 120,
    "mapCoverUrl": "https://...",
    "matchDuration": 600,
    "likeCount": 450,
    "gameMode": 2,
    "mapCode": "ABCD-1234"
  }
}
```
**Erros**: 400 `missing map_code or region|region_not_implemented`, 404 `map_not_found`.

---

## 15) **Revogar token** (logout Garena)
**GET** `https://freefirefwx-beta.squareweb.app/api/revoke_token?access_token=<game_token>`

**Respostas**
- **200**
```json
{ "status": "success", "message": "token revoked successfully!" }
```
- **400** `token already revoked or invalid | invalid token format`
- **500** `failed to verify token | error during token revocation process`

---

## 16) **Check ban**
**GET** `https://freefirefwx-beta.squareweb.app/api/check_ban?id=<uid>`

**Resposta (200)**
```json
{
  "message": "OK",
  "details": {
    "PlayerNickname": "JogadorX",
    "PlayerRegion": "BR",
    "is_banned": "no",
    "banned_period": 0
  }
}
```
**Notas**: `/api/check_banned` está **obsoleto** (placeholder). Use **`/api/check_ban`**.

---

## 17) **Versão do jogo/servidor**
**GET** `https://freefirefwx-beta.squareweb.app/api/version`

**Resposta (200)**
```json
{
  "status": "success",
  "serverStatus": {
    "version": "1.114.2",
    "isServerOpen": true,
    "isReviewServer": false
  },
  "ffRelease": {
    "releaseVersion": "OB50",
    "serverUrl": "https://...",
    "version": "1.114.1"
  }
}
```
**Erros**: 502/503 quando a API externa de versão falha.

---

## 18) **Apelido pelo UID** (resolução rápida)
**GET** `https://freefirefwx-beta.squareweb.app/api/nick?id=<uid>&region=<opcional>`

- Sem `region`, tenta automaticamente nas regiões configuradas.
- **Resposta (200)**
```json
{ "player_nickname": "JogadorX", "player_region": "BR" }
```
**Erros**: 400 `missing id parameter|invalid id parameter`, 404 `player_not_found`.

---

## 19) **Diamantes / Carteira**
**GET** `https://freefirefwx-beta.squareweb.app/api/diamonds?region=<br|...>&(access_token|jwt|uid&password)`

**Resposta (200)**
```json
{
  "success": true,
  "message": "Diamond information successfully obtained!",
  "accountDiamondsBasicInfo": {
    "nickname": "JogadorX",
    "diamonds": 1234,
    "gold": 98765,
    "last_recharge": 1719999999,
    "total_diamonds": 45678
  }
}
```
**Erros**: 400 parâmetros ausentes ou `region_not_implemented`; 401 `INVALID_TOKEN|INVALID_GUEST`; 500 `failed_to_fetch_account_details`.

---

## 20) **Estatísticas (CS / BR)**
**GET** `https://freefirefwx-beta.squareweb.app/api/stats?id=<uid>&region=<br|...>&mode=<cs|br>`

**Resposta (200) – exemplo `mode=cs`**
```json
{
  "uid": "12345678",
  "results": {
    "CS_CAREER": {
      "Games": 1000,
      "Wins": 520,
      "Eliminations": 3000,
      "Deaths": 2500,
      "KdRatio": "1.20",
      "Headshots": 800,
      "HeadshotRate": "26.67%",
      "AvgDamagePerMatch": 850.4,
      "WinRate": "52.00%",
      "MVP": 123,
      "Assist": 400,
      "Knockdowns": 900,
      "TripleTakedown": 10,
      "QuadraTakedown": 2,
      "TotalDamage": 850400
    },
    "CS_CLASSIC": { "...": "..." },
    "CS_RANKED":  { "...": "..." }
  }
}
```
**Para `mode=br`** chaves são `BR_CAREER`, `BR_CLASSIC`, `BR_RANKED` com métricas equivalentes.

**Erros**: 400 `invalid_request|invalid_id|region_not_implemented`.

---

## 21) **Serviço de Imagens (avatar/roupas)**
**GET** `https://freefirefwx-beta.squareweb.app/api/v1/ff/<avatar|clothes>/<png|webp|jpeg>?image=<hash_dir>/<arquivo>`

- Retorna o **arquivo** da imagem (200) ou:
  - 400 `{ "error": "Invalid request" }`
  - 404 `{ "error": "File not found" }`

---

## Autenticação & prioridades

Alguns endpoints aceitam múltiplas formas:
- `access_token` (game token) → a API obtém um **JWT** internamente.
- `jwt` direto (se já tiver).
- `uid` + `password` (guest) → a API busca `access_token` e então o JWT.

Se mais de um for enviado, a ordem preferida é **access_token** → **jwt** → **uid/password**.

---

## Convenções de erro (resumo)

- **400**: parâmetros ausentes/invalid (`invalid_request`, `invalid_id`, `invalid_guild_id`, `region_not_implemented`, `AccessToken invalid`).
- **401**: autenticação inválida (`INVALID_TOKEN`, `INVALID_GUEST`).
- **404**: recurso não encontrado (`player_not_found`, `guild_not_found`, `no players found`, arquivos de imagem).
- **5xx**: falhas internas/externas (`failed_to_parse_response`, `failed_to_connect`, `internal server error`).

---

## Observações

- Campos seguem exatamente os nomes dos *protos* quando convertido em JSON.
- Para ampliar regiões, edite/adicione contas em `accounts.json` com `covers_regions`.
- Imagens de avatar/banners e roupas são persistidas em `AVATARS/` e `CLOTHES/` e servidas pelo endpoint `/api/v1/ff/...`.
