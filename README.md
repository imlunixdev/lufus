<div align="center">
  <img width="190" alt="Logo do Lufus: Tux com um pendrive" src="https://github.com/user-attachments/assets/49b201df-d91e-4450-8047-25a6a02fbb02" />
  <h1>Lufus</h1>
  <p><strong>Linux USB Formatting Utility, with Source</strong></p>
  <p>Grave imagens, prepare instaladores e formate unidades USB no Linux.</p>
  <p><code>Rust</code> · <code>GTK 4</code> · <code>MIT</code> · <code>0.1.0-alpha</code></p>
</div>

<p align="center">Ferramenta gráfica para Linux, com operações de disco autorizadas pelo PolicyKit.</p>

---

## Visão geral

O Lufus reúne gravação de imagens, criação de mídia de instalação do Windows, formatação e apagamento de unidades em uma interface GTK 4. O projeto é escrito em Rust e solicita autorização administrativa apenas quando uma operação precisa modificar um disco.

> **Atenção:** gravar, formatar ou apagar uma unidade remove dados. Confira o dispositivo e a capacidade na tela de confirmação antes de iniciar.

## Recursos

- **Gravação de ISO e IMG:** grava a imagem diretamente no dispositivo e mostra o progresso. A tabela de partições e os sistemas de arquivos definidos pela própria imagem são preservados.
- **Verificação de ISO:** identifica inicialização por BIOS e/ou UEFI. Quando as ferramentas e a rede estão disponíveis, também tenta identificar o kernel e compará-lo com o catálogo estável do kernel.org. Essa verificação não autentica a imagem.
- **Mídia de instalação do Windows:** prepara uma unidade FAT32 inicializável por UEFI a partir de uma ISO x64 ou ARM64. Imagens de instalação maiores que o limite de arquivo do FAT32 são divididas em partes compatíveis.
- **Opções do Windows:** permite selecionar requisitos do Windows 11 para ignorar ou fornecer um `Autounattend.xml` personalizado. Essas opções são alternativas; o XML escolhido não é combinado com o bypass.
- **Formatação:** cria tabela de partições MBR ou GPT e formata como FAT32, exFAT ou NTFS, com rótulo, tamanho de cluster e opção rápida ou completa.
- **Apagamento:** sobrescreve o dispositivo inteiro com zeros. A operação pode demorar e não pode ser desfeita.
- **Proteção contra seleção acidental:** unidades fixas ficam ocultas inicialmente. A opção avançada para listá-las deve ser usada com cuidado.
- **Idiomas:** Português do Brasil, Português, English, Español, Русский, 日本語, 中文（普通话） e العربية. A preferência é salva em `~/.config/lufus/language`.

## Requisitos

Para compilar, instale Rust/Cargo, um compilador C, `pkg-config` e os arquivos de desenvolvimento do GTK 4.

As operações de disco também usam ferramentas do sistema, incluindo `pkexec`/PolicyKit, `sfdisk`, `mount` e `umount`. Formatar exige o formatador do sistema escolhido: `dosfstools` para FAT32, `exfatprogs` para exFAT ou `ntfs-3g` para NTFS.

Recursos opcionais:

- Preparação de mídia Windows e leitura da versão da ISO: `wimlib-imagex` (fornecido pelo pacote `wimlib` ou equivalente da distribuição).
- Verificação mais detalhada de ISO: `bsdtar`, `file` e `curl`. Sem essas ferramentas ou sem rede, a gravação de imagens continua disponível; a consulta ao kernel.org pode ser omitida.

Exemplos de dependências para compilar e usar as operações básicas:

**Arch Linux**

```sh
sudo pacman -S --needed rust cargo base-devel pkgconf gtk4 util-linux polkit dosfstools exfatprogs ntfs-3g
```

**Debian / Ubuntu**

```sh
sudo apt install cargo rustc build-essential pkg-config libgtk-4-dev util-linux policykit-1 dosfstools exfatprogs ntfs-3g
```

**Fedora**

```sh
sudo dnf install cargo rust gcc pkgconf-pkg-config gtk4-devel util-linux polkit dosfstools exfatprogs ntfs-3g
```

## Executar a partir do código

Na pasta do projeto:

```sh
sh Lufus.sh
```

O lançador compila os binários em `/tmp/lufus-build-<UID>` e abre o aplicativo. Execute o Lufus como usuário normal, **sem `sudo`**; o PolicyKit apresenta a autenticação quando uma operação de disco precisar dela.

Para compilar manualmente:

```sh
cargo build --locked --release --bins --target-dir "/tmp/lufus-build-$(id -u)/target"
```

## Instalar no sistema

Depois de compilar no diretório usado pelo `Makefile`, instale com:

```sh
sudo make install TARGET_DIR="/tmp/lufus-build-$(id -u)/target"
```

O destino padrão é `/usr`. A instalação inclui os executáveis, a logo/ícone, o atalho de aplicativos e a política do PolicyKit. Para remover os mesmos arquivos:

```sh
sudo make uninstall
```

## Como as operações funcionam

### Gravar ISO ou IMG

O arquivo é copiado em modo raw para o disco selecionado. O Lufus não altera a imagem para escolher outro sistema de arquivos ou esquema de partição. A imagem precisa caber no destino.

### Preparar Windows

O Lufus verifica a estrutura da ISO, prepara uma partição FAT32 e copia os arquivos para criar mídia de instalação UEFI. Se `install.wim` for grande demais para FAT32, ele é dividido; um `install.esd` grande é convertido antes da divisão. Um `Autounattend.xml` personalizado é validado e copiado para a raiz da unidade. O bypass do Windows 11 pode permitir a instalação em hardware que não cumpra os requisitos selecionados.

### Formatar ou apagar

Formatação rápida recria a tabela e o sistema de arquivos sem zerar previamente toda a partição. A formatação completa zera a partição criada e depois a formata. **Apagar disco** grava zeros em todo o dispositivo e deixa a unidade sem tabela de partições.

## Estrutura do projeto

```text
.
├── data/                 # Atalho do aplicativo e política do PolicyKit
├── src/
│   ├── main.rs           # Interface, inspeção de ISO e fluxo do aplicativo
│   ├── worker.rs         # Gravação, preparação, formatação e apagamento
│   └── i18n.rs           # Idiomas da interface e mensagens
├── .github/workflows/    # Build e pacote Linux para releases
├── Lufus.sh              # Compilação e execução a partir do código
├── LufusLogo.png         # Logo e ícone
├── Makefile              # Build, instalação e remoção
└── Cargo.toml            # Pacote Rust e versão
```

## Releases

O workflow em `.github/workflows/release.yml` compila um pacote Linux x86_64 quando uma tag `v*` é enviada. O workflow também pode ser iniciado manualmente para gerar um artefato de build. Consulte o último release para as instruções e o resumo da versão atual.

## Licença

O código é distribuído sob a licença MIT. Consulte [`LICENSE`](../LICENSE).
