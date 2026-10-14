# Kepleræ macOS 2.0

Aplicativo nativo para macOS 26 ou superior, com interface SwiftUI e Liquid Glass. A compilação gera um aplicativo universal para Apple Silicon e Intel, um ZIP e um instalador DMG.

## Recursos

- Barra lateral para Início, Pomodoro, Questões, Notas e Perfil.
- Catálogo original da versão 2.0: 1.022 materiais (703 aulas, 306 PDFs e 13 áudios).
- Ordem de Português preservada; links de pastas do Drive resolvem o arquivo da aula pelo nome.
- Reprodução com AVPlayer, leitor PDFKit com janela em tela cheia e cache local.
- Pomodoro com animações transparentes, sons ambientes, notas e histórico de questões.
- Atalhos ⌘1–4 para as seções, ⌘, para Perfil, além dos atalhos de edição.
- Mesmo contrato de sincronização Android/iOS: `keplerae-sync-v1.json`, schema 2, Google Drive appDataFolder. Envio automático com debounce, consulta a cada 20 segundos enquanto ativo e consulta ao retornar ao aplicativo.

## Compilar pelo GitHub

O workflow **Compilar Kepleræ macOS 2.0** executa ao enviar alterações para `main` ou manualmente em **Actions → Run workflow**. Ele valida os recursos, executa os testes de compatibilidade do schema, compila as duas arquiteturas, verifica a abertura do aplicativo e disponibiliza o artefato **Keplerae-macOS-2.0-universal**.

O artefato contém `Keplerae-macOS-2.0.dmg`, `Keplerae-macOS-2.0.zip`, os hashes SHA-256 e uma captura da abertura no runner. Abra o DMG e arraste Keplerae para Aplicativos.

Se você recebeu os fontes em ZIP, há duas maneiras de enviar ao repositório:

1. Extraia o ZIP e envie todos os arquivos, incluindo `.github/workflows/build-macos.yml`, mantendo as pastas.
2. Envie `Keplerae-macOS-2.0-fontes.zip` para a raiz e crie o arquivo `.github/workflows/build-macos.yml` com o conteúdo do workflow fornecido. O workflow extrai os fontes antes de compilar.

### Login Google e assinatura Apple

Sem certificado configurado, o workflow gera um **build de teste com assinatura ad hoc**, sem notarização. Esse pacote permite conferir a compilação e a abertura; **o login Google não deve ser considerado funcional nesse build**. O SDK exige assinatura emitida pela Apple e permissão de acesso ao Chaves para guardar a sessão no macOS.

Para gerar o pacote com assinatura Developer ID Application, configure estes **Secrets** do repositório:

| Secret | Conteúdo |
| --- | --- |
| `MACOS_CERTIFICATE_P12_BASE64` | Certificado Developer ID Application e chave privada exportados como P12 e codificados em base64 |
| `MACOS_CERTIFICATE_PASSWORD` | Senha do arquivo P12 |
| `MACOS_PROVISION_PROFILE_BASE64` | Perfil de distribuição Developer ID para o bundle ID macOS, autorizando o grupo do Chaves, codificado em base64 |
| `APPLE_TEAM_ID` | Identificador da equipe Apple Developer |

Para também notarizar o app, configure `APPLE_ID` e `APPLE_APP_SPECIFIC_PASSWORD`. Não envie certificados, senhas ou chaves para os arquivos do repositório. O workflow importa o certificado em um chaveiro temporário, assina o aplicativo e remove o chaveiro ao finalizar.

O grupo principal do Chaves deve ser `$(AppIdentifierPrefix)$(CFBundleIdentifier)`. O workflow incorpora o perfil e deriva o prefixo dele, validando equipe, bundle ID, grupo autorizado e validade antes de assinar.

### Cliente OAuth

- Projeto Google compartilhado: `240965240632`.
- Client ID público macOS: `240965240632-qd0sekmr9ukauuh7lnc2cmct75ja5grf.apps.googleusercontent.com`.
- Bundle ID macOS: `com.aistudio.keplerae.stdy.macos`.
- URL scheme: `com.googleusercontent.apps.240965240632-qd0sekmr9ukauuh7lnc2cmct75ja5grf`.

No Google Cloud, confirme que o client ID reservado é do tipo **iOS**, como exige o SDK nativo também no macOS, e está registrado para o bundle ID acima. Não há client secret no aplicativo. O consentimento deve permitir `drive.readonly`, `drive.file` e `drive.appdata`, com a mesma conta usada no Android/iPhone.

Referências oficiais: [configuração Google Sign-In para iOS/macOS](https://developers.google.com/identity/sign-in/ios/start-integrating) e [armazenamento de credenciais macOS no GTMAppAuth](https://github.com/google/GTMAppAuth#macos-keychain-storage).

## Compilar localmente

Instale Xcode com SDK macOS 26 e XcodeGen (`brew install xcodegen`). Execute:

```sh
bash scripts/build_macos.sh
```

O projeto Xcode é gerado a partir de `project.yml`. Para assinar localmente, defina `MACOS_SIGN_IDENTITY` com o nome do certificado Developer ID Application no Chaves, `APPLE_TEAM_ID` e `MACOS_PROVISION_PROFILE` com o caminho do perfil. Os pacotes ficam em `dist/`.

## Verificações

`scripts/validate_macos.py` confere catálogo, recursos, ícones e configuração OAuth. `Tests/CloudStateTests.swift` testa migração de timestamps Android em segundos/milissegundos, valores padrão e ida e volta de notas, favoritos, aulas concluídas, questões, histórico Pomodoro e preferências.

Abertura e compilação não substituem o teste com conta Google: após assinar, confirme login, recebimento dos dados já existentes e propagação de alterações entre Mac, Android e iPhone. A regra de resolução de conflitos é a mesma da 2.0: o snapshot com data mais recente prevalece; edições simultâneas em dispositivos diferentes não são mescladas por campo.

## Origem

Portado do pacote `Keplerae-iOS-2.0.zip` fornecido no backup de continuidade 2.0. Os projetos Android e iOS existentes não são modificados por este repositório.
