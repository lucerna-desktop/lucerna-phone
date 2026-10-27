# Proposta do lucerna-phone

O `lucerna-phone` gerencia celulares a partir do desktop.

- **Espelhamento e controle:** pelo [scrcpy](https://github.com/Genymobile/scrcpy).
- **Demais funções:** recria em Rust as outras funções do KDE Connect, com um protocolo próprio e um app próprio no celular, o [`lucerna-phone-android`](https://github.com/lucerna-desktop/lucerna-phone-android).
- **Uso:** funciona sozinho, com uma TUI e uma linha de comando, em qualquer ambiente e com ou sem systemd. Com o Lucerna, ganha um painel.

Segue a estrutura de apps do [ecossistema](https://github.com/lucerna-desktop/lucerna-sdk/blob/main/docs/ecossistema.md): núcleo, serviço, TUI e painel em QML.

## Painel e TUI

O painel serve para olhar e agir rápido. O trabalho completo fica na TUI. Nenhuma ação existe só no painel: tudo o que ele faz também se faz pela TUI e pela linha de comando.

**No painel:**

- o aparelho: bateria, sinal e se está conectado;
- ações de um clique: espelhar a tela pelo scrcpy, enviar um arquivo, tocar o celular para achá-lo e copiar a área de transferência;
- as últimas notificações do celular, com resposta a mensagens.

**As mesmas ações rápidas, sem o painel:**

- **na TUI:** o estado do aparelho, as quatro ações e a lista das notificações do celular, com resposta a mensagens;
- **na linha de comando:** `lucerna-phone mirror`, `lucerna-phone send <arquivo>`, `lucerna-phone ring`, `lucerna-phone clipboard`, `lucerna-phone notifications` (lista) e `lucerna-phone reply <notificação> "texto"`; com mais de um aparelho, `--device <nome>`. Os nomes finais ficam para o plano de implementação.

**Notificações do celular:** vão para o servidor de notificações do sistema (`org.freedesktop.Notifications`), que o shell atende e que o KDE e o GNOME também atendem; não dependem do shell. A resposta a mensagens vem como ação da notificação: onde o servidor oferece resposta direta no próprio aviso (o KDE oferece), ela é usada; onde não oferece (o GNOME, por exemplo), a ação "Responder" abre a TUI já na conversa. Sem servidor de notificações nenhum, elas ficam só na lista da TUI e do painel.

**Só na TUI:**

- parear;
- escolher o que sincronizar;
- navegar pelos arquivos do celular;
- permissões.

## Em aberto

- Quais funções do KDE Connect entram primeiro.
- A descoberta e o pareamento.
- A segurança e a criptografia.
- Android só ou também iOS.
- Como a conexão com o celular fica ouvindo. O celular precisa achar o computador e mandar notificações, então alguém tem que estar ouvindo a porta, e o serviço sob demanda do [ecossistema](https://github.com/lucerna-desktop/lucerna-sdk/blob/main/docs/ecossistema.md) não cobre isso sozinho. Duas opções, sem decisão ainda:
  - **Ativação por socket de rede do systemd:** o systemd ouve a porta e sobe o serviço quando o celular conecta. O serviço continua sob demanda.
  - **Serviço sempre ligado só enquanto houver aparelho pareado:** sem aparelho pareado, ele volta a ser sob demanda.

  **Sem systemd, vale a segunda opção, qualquer que seja a escolhida com systemd:** com aparelho pareado, o serviço fica ligado ouvindo a porta (um aparelho pareado conta como trabalho pendente, então ele não sai por ociosidade). Se ele não estiver rodando, qualquer cliente (TUI, linha de comando, painel) o sobe, como em todo app do ecossistema. Para ele já estar ouvindo depois do login, o app instala, enquanto houver aparelho pareado, uma entrada de início automático do XDG (`~/.config/autostart/lucerna-phone.desktop`, com `lucerna-phone serve`), que o KDE, o GNOME e o XFCE leem; no Hyprland sem systemd, a dica é `exec-once = lucerna-phone serve` no `hyprland.conf`. **Limite:** sem systemd e sem nada que suba o serviço no login, o celular só acha o computador depois que o usuário abrir a TUI, a linha de comando ou o painel.
