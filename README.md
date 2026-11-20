# Kepleræ macOS 2.0.1

Aplicativo nativo para macOS 26 ou superior, com interface SwiftUI e Liquid Glass. A compilação gera um aplicativo universal para Apple Silicon e Intel, um ZIP e um instalador DMG.

## Versão 2.0.1 (build 21)

Corrige o movimento dos ponteiros do Pomodoro usando o tempo decorrido e ângulos contínuos, inclusive na virada do minuto. O visor continua em contagem regressiva. Remove o painel externo indicado na referência e oferece os acabamentos Porcelana, Grafite, Argila, Azul profundo e Sinal.

## Recursos

- Barra lateral para Início, Pomodoro, Questões, Notas e Perfil. Início limpa a navegação de matérias, pastas e materiais abertos.
- Cards animados com seis cores, 289,8 pontos de altura e os padrões 5×5 da referência Pattern Cards: círculos/contornos, partículas, interação com o cursor e brilho. Os cabeçalhos de matérias e pastas reutilizam o efeito em uma área separada do texto.
- Seletor de tema apenas na barra lateral, desenhado em Canvas para não depender do material nativo durante a troca de aparência.
- Catálogo original da versão 2.0: 1.022 materiais (703 aulas, 306 PDFs e 13 áudios).
- Ordem de Português preservada; links de pastas do Drive resolvem o arquivo da aula pelo nome.
- Reprodução com AVPlayer, leitor PDFKit com janela em tela cheia e cache local.
- Pomodoro com três ponteiros em sentido horário, mostrador deslocado e cinco acabamentos inspirados em Dial Finishes. Painel colorido externo removido; sons ambientes e histórico preservados.
- Atalhos ⌘1–4 para as seções, ⌘, para Perfil, além dos atalhos de edição.
- Mesmo contrato de sincronização Android/iOS: `keplerae-sync-v1.json`, schema 2, Google Drive appDataFolder. Envio automático com debounce, consulta a cada 20 segundos enquanto ativo e consulta ao retornar ao aplicativo.

## Compilar pelo GitHub

O workflow **Compilar Kepleræ macOS** executa ao enviar alterações para `main` ou manualmente em **Actions → Run workflow**. Ele valida os recursos, executa os testes de compatibilidade do schema, compila as duas arquiteturas, verifica a abertura do aplicativo e disponibiliza o artefato **Keplerae-macOS-universal**.

O artefato contém `Keplerae-macOS-2.0.1.dmg`, `Keplerae-macOS-2.0.1.zip`, os hashes SHA-256 e uma captura da abertura no runner. Abra o DMG e arraste Keplerae para Aplicativos.

Se você recebeu os fontes em ZIP, há duas maneiras de enviar ao repositório:

1. Extraia o ZIP e envie todos os arquivos, incluindo `.github/workflows/build-macos.yml`, mantendo as pastas.
2. Envie `Keplerae-macOS-2.0-fontes.zip` para a raiz e crie o arquivo `.github/workflows/build-macos.yml` com o conteúdo do workflow fornecido. O workflow extrai os fontes antes de compilar.

### Login Google e assinatura Apple

O login usa o fluxo oficial público do Google com AppAuth: navegador do sistema, PKCE, validação do retorno e atualização dos tokens. GTMAppAuth guarda as credenciais no Chaves local do Mac (`useFileBasedKeychain`), com controle de acesso por aplicativo. O login não depende de assinatura Apple paga nem de grupos restritos do Chaves. O aplicativo não abre pedidos automáticos de senha do Chaves: consultas e gravações são não interativas e mantêm as ACLs existentes. Se uma atualização ad hoc não tiver acesso à credencial protegida por outra assinatura, será necessário entrar novamente no Google. Se o Chaves também impedir a gravação, a sessão funciona apenas até fechar o app e o Perfil informa essa condição. Para preservar a identidade entre atualizações, distribua builds assinadas com o mesmo certificado Developer ID.

Sem certificado configurado, o workflow gera um **build de teste com assinatura ad hoc**, sem notarização. Para distribuição com assinatura Developer ID Application, configure estes **Secrets** do repositório:

| Secret | Conteúdo |
| --- | --- |
| `MACOS_CERTIFICATE_P12_BASE64` | Certificado Developer ID Application e chave privada exportados como P12 e codificados em base64 |
| `MACOS_CERTIFICATE_PASSWORD` | Senha do arquivo P12 |

Para também notarizar o app, configure `APPLE_TEAM_ID`, `APPLE_ID` e `APPLE_APP_SPECIFIC_PASSWORD`. Não envie certificados, senhas ou chaves para os arquivos do repositório. O workflow importa o certificado em um chaveiro temporário, assina o aplicativo e remove o chaveiro ao finalizar.

### Cliente OAuth

- Projeto Google compartilhado: `240965240632`.
- Client ID público macOS: `240965240632-g58uffmvvthdqa1mk513fd4k8m8jml2v.apps.googleusercontent.com`.
- Bundle ID macOS: `com.aistudio.keplerae.stdy.macos`.
- URL scheme: `com.googleusercontent.apps.240965240632-g58uffmvvthdqa1mk513fd4k8m8jml2v`.

No Google Cloud, confirme que o client ID reservado é do tipo **iOS**, como exige o SDK nativo também no macOS, e está registrado para o bundle ID acima. Não há client secret no aplicativo. O consentimento deve permitir `drive.readonly`, `drive.file` e `drive.appdata`, com a mesma conta usada no Android/iPhone.

Referências oficiais: [OAuth para aplicativos nativos](https://developers.google.com/identity/protocols/oauth2/native-app), [AppAuth](https://github.com/openid/AppAuth-iOS) e [armazenamento macOS no GTMAppAuth](https://github.com/google/GTMAppAuth#macos-keychain-storage).

## Compilar localmente

Instale Xcode com SDK macOS 26 e XcodeGen (`brew install xcodegen`). Execute:

```sh
bash scripts/build_macos.sh
```

O projeto Xcode é gerado a partir de `project.yml`. Para assinar localmente, defina `MACOS_SIGN_IDENTITY` com o nome do certificado Developer ID Application no Chaves. Os pacotes ficam em `dist/`.

## Verificações

`scripts/validate_macos.py` confere catálogo, recursos, ícones e configuração OAuth. `Tests/CloudStateTests.swift` testa migração de timestamps Android em segundos/milissegundos, valores padrão e ida e volta de notas, favoritos, aulas concluídas, questões, histórico Pomodoro e preferências.

Abertura e compilação não substituem o teste com conta Google: após assinar, confirme login, recebimento dos dados já existentes e propagação de alterações entre Mac, Android e iPhone. A regra de resolução de conflitos é a mesma da 2.0: o snapshot com data mais recente prevalece; edições simultâneas em dispositivos diferentes não são mescladas por campo.

## Parâmetros visuais

A referência é [Pattern Cards, de Zsolt Kacso](https://experiments.thisiswhitespace.com/pattern-cards). O app usa um port nativo em SwiftUI/Canvas; não carrega scripts remotos nem incorpora uma WebView. Os 51 parâmetros solicitados ficam em `Resources/PatternCards.defaults.json`. A altura foi posteriormente reduzida em 30%, de 414 para 289,8, conforme solicitado. Os demais valores foram mantidos.

`Prototype/PatternCards.useDialKit.ts` contém a chamada `useDialKit("Pattern Cards", ...)` com os mesmos defaults para uso no protótipo web. Esse arquivo é uma referência de configuração e não uma dependência do app macOS.

Os testes cobrem a paridade dos defaults, a geometria dos três padrões, o crescimento e retorno dos círculos, os limites de partículas, a limpeza da navegação e a restauração do controle de interação do Chaves em caso de erro.

## Origem

Portado do pacote `Keplerae-iOS-2.0.zip` fornecido no backup de continuidade 2.0. Os projetos Android e iOS existentes não são modificados por este repositório.
