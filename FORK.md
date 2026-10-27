# Fork local de `@loserfox/telegram`

Copia del repo original (`https://github.com/loserfox/telegram`, commit de
`main` del 2026-10-26) con cuatro arreglos y una opción nueva. El objetivo es
que los parches **sobrevivan a reinstalaciones**: se instalan desde esta carpeta
(o desde tu fork en GitHub), no editando `node_modules`.

## Cambios respecto a 0.1.0

| Fichero | Cambio | Motivo |
|---|---|---|
| `src/bridge.ts` + `lib/bridge.js` | `ensureChat` crea una sesión nueva (`telegram:<chat>:<epoch>`, igual que `/new`) con timeout de 45 s | Reanudar un id de sesión ya existente dejaba `ctx.agents.create` colgado: el bucle moría y el bot no respondía hasta reiniciar |
| `src/bridge.ts` + `lib/bridge.js` | Timeout de 120 s por update | Un update atascado ya no puede detener el long polling |
| `src/bridge.ts` + `lib/bridge.js` | Log de diagnóstico en `<perfil>/telegram-debug.log` | El logger del runtime no es visible desde la app de escritorio |
| `src/client.ts` + `lib/client.js` | `AbortSignal.timeout(120000)` por llamada a la Bot API | Un socket colgado falla y se reintenta en vez de congelar el bucle |
| `src/index.ts` + `lib/index.js` | Opción de config `cwd` (vacía = `process.cwd()`) | Los agentes de Telegram trabajan en el workspace, no en el directorio del perfil |
| `package.json` | versión `0.1.1`; peers alineados con el runtime `0.2.1-rc.3` / `cordis 4.0.5` / `schemastery 3.18.5` | Evita depender de la exención de compatibilidad de DSH |

`lib/` se sincronizó a mano con `src/` (el `scripts/build.sh` original necesita
un checkout de dsh con `tsc`). Si vas a mantener el fork, reconstruye con ese
script y comprueba que el diff sigue siendo el de arriba.

## Instalar

```powershell
# desde esta carpeta
dsh plugin --profile desktop add "D:\Vic Val\Documentos\deepseek-harness\default-workspace\telegram-fork"

# o, tras subirlo a tu GitHub
dsh plugin --profile desktop add "github:<tu-usuario>/telegram"
```

Después hay que **reiniciar DeepSeek Harness** (cambios de composición y de
código del plugin no entran por HMR).

Config en `~/.dsh/profiles/desktop/cordis.patch.yml`:

```yaml
- id: telegram
  config:
    token: "<token de @BotFather>"
    allowedUserIds: [632378130]
    provider: deepseek-account
    model: deepseek-flash
    cwd: 'D:\Vic Val\Documentos\deepseek-harness\default-workspace'
```

## Limitaciones que siguen ahí

- Cada reinicio abre una sesión nueva por chat (consecuencia del arreglo de
  `ensureChat`): el bot no continúa la conversación anterior.
- Sin watchdog: si el runtime descarga la fibra del plugin, nada lo rearma hasta
  reiniciar la app. El log permite ver cuándo pasó.
- Solo chats privados y solo texto. Token en texto plano en el perfil.
- Al subir de versión de DSH habrá que revisar los `peerDependencies`.
