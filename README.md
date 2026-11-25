# Fpsff
Acelerador de jogos
 Booster – Game Smooth

App Android (Kotlin + Jetpack Compose, minSdk 26, sem root) que monitora o desempenho enquanto você joga:
contador flutuante, FPS mín/máx/médio, temperatura, RAM, histórico, identificação de jogos instalados,
perfis por jogo e três modos (Gráficos Suaves, Desempenho Máximo, Equilibrado) com visual inspirado em blocos.

> **Status honesto:** este projeto foi escrito sem acesso a um Android SDK, então **ainda não foi compilado nem
> testado em aparelho**. Pode haver pequenos erros de compilação a ajustar na primeira vez. O APK precisa ser gerado
> por você (3 caminhos abaixo, o mais simples é o Android Studio).

## Gerar o APK

**A) Android Studio (recomendado)**
1. Instale o Android Studio (versão Ladybug ou mais nova).
2. `File > Open` e escolha a pasta `FPSBooster-GameSmooth`. Aguarde o Gradle sincronizar (baixa o SDK 35 se faltar).
3. `Build > Build Bundle(s) / APK(s) > Build APK(s)`.
4. O arquivo fica em `app/build/outputs/apk/debug/app-debug.apk`.

**B) Linha de comando** (JDK 17, Android SDK 35 e Gradle 8.9 instalados)
```
gradle :app:assembleDebug
```

**C) GitHub Actions (sem instalar nada)**
1. Suba esta pasta para um repositório seu no GitHub.
2. Aba *Actions > Build APK > Run workflow*.
3. Baixe o artefato `FPS-Booster-Game-Smooth-APK` (contém o `app-debug.apk`).

O APK de debug já vem assinado com a chave de debug e é instalável direto.

## Instalar no celular
1. Copie o `app-debug.apk` para o aparelho.
2. Abra o arquivo e permita "Instalar apps desconhecidos" para o app que você usou para abri-lo.
3. Instale e abra **FPS Booster – Game Smooth**.

## Primeira configuração
1. Aba **INÍCIO**: conceda **Acesso de uso** (para detectar qual app está aberto) e **Sobreposição** (contador flutuante). Permita notificações.
2. Aba **JOGOS**: toque num jogo para escolher o modo, ligar/desligar o monitoramento e o overlay.
3. Toque em **INICIAR MONITORAMENTO** e abra o jogo. O contador aparece sobre ele (arraste para mover).
4. Aba **HISTÓRICO**: sessões com mín/máx/média de FPS, temperatura, RAM e gráfico.

## FPS real do jogo (opcional, sem root)
O Android **não permite** que um app comum leia o FPS de outro app. Existe um caminho oficial de
desenvolvedor: a permissão `DUMP`, que você concede uma vez via ADB:
```
adb shell pm grant com.fpsbooster.gamesmooth android.permission.DUMP
```
Com ela, o app lê os tempos de apresentação de quadros do jogo no SurfaceFlinger (`dumpsys SurfaceFlinger --latency`).
O formato dessa saída varia entre fabricantes/versões; a escolha da camada do jogo é heurística.

**Sem a permissão (ou se não funcionar no seu aparelho)** o app mostra o **FPS da TELA** (Choreographer), sempre
rotulado como "não é o FPS do jogo". Nenhum valor é simulado: o que não estiver disponível aparece como `--`.

## Contador flutuante estilo massinha
O overlay é um painel arredondado com gráfico de linha grossa e brilho (30 últimos valores reais de FPS).
É ultra leve: sem imagens nem animações, só um traçado redesenhado 1 vez por segundo. A cor da linha muda
(verde, amarelo, vermelho) conforme o FPS. Ele aparece por cima do jogo usando a sobreposição oficial do Android;
**não há injeção no jogo** e o app não modifica o processo nem os arquivos dele.

## O que cada dado significa
| Dado | Fonte | Observação |
|---|---|---|
| FPS do jogo | SurfaceFlinger (requer `DUMP` via ADB) | Real, mas depende do aparelho |
| FPS da tela | `Choreographer` | Não é o FPS do jogo |
| Temperatura | `BatteryManager` | É a da **bateria**; CPU/GPU não são expostas |
| Estado térmico | `PowerManager` (Android 10+) | Informado pelo sistema |
| RAM | `ActivityManager.MemoryInfo` | Uso da memória do sistema |
| Jogo em primeiro plano | `UsageStatsManager` | Requer Acesso de uso |
| Jogos instalados | `PackageManager` (categoria de jogo) | Use "Mostrar todos" se faltar algum |

## Os modos
Não alteram arquivos nem configurações internas dos jogos e não "criam" FPS. Cada modo define alerta de temperatura,
estilo do overlay e dicas para você aplicar no jogo. O **Desempenho Máximo** também pede ao Android
(`killBackgroundProcesses`) que encerre apps não-sistema em segundo plano ao abrir o jogo; o sistema pode reabri-los.

## Privacidade
Sem permissão de internet, anúncios ou contas. Tudo fica no aparelho.

## Estrutura
```
app/src/main/java/com/fpsbooster/gamesmooth/
  MainActivity.kt
  model/Models.kt            modos, perfis, sessões
  data/Store.kt              perfis e histórico (JSON local)
  system/                    permissões, scanner de jogos, app em primeiro plano, RAM/temperatura
  monitor/                   serviço, overlay, fontes de FPS
  ui/                        tema em blocos e telas (Compose)
```
Ícone e paleta próprios, inspirados em blocos; sem assets, fontes ou marcas da Mojang/Microsoft.
