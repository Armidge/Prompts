# PCAP/PCAPNG Automated Triage Prompt

Este prompt para LLMs com execução de código automatiza a triagem técnica de arquivos de captura de rede (`.pcap`/`.pcapng`). Sem exigir configurações complexas além do envio do arquivo, a ferramenta utiliza utilitários como `capinfos`, `tshark` e `scapy` para garantir que todas as métricas do relatório venham de dados reais, e não de estimativas.

O processo de análise segue cinco etapas estruturadas: resumo da captura (duração, volume e pares de IPs), hierarquia de protocolos, avaliação de saúde de transporte e sessão, detecção de ameaças (como port scan, força bruta, tunelamento e credenciais em texto claro) e diagnóstico diferencial de latência entre rede e aplicação.

Para evitar alucinações e dados inventados, o modelo sinaliza informações indisponíveis com termos padronizados, valida artefatos de captura e atribui níveis de confiança aos achados. A entrega final traz um veredito conciso de três linhas, tabelas em Markdown e os filtros do Wireshark específicos para cada fluxo suspeito identificado.

## Prompt

```text
Analise o arquivo .pcap/.pcapng anexado. Use capinfos e tshark (se não estiverem disponíveis, use scapy) e execute os comandos: todo número deve vir da saída de uma ferramenta, nunca de estimativa. Se algo não puder ser determinado, diga "não determinável" e explique o motivo.

Faça a triagem nesta ordem:

1.: Resumo: duração, total de pacotes e bytes, taxa média em bits/s e pacotes/s, e os 5 principais pares de IPs por bytes.
2.: Protocolos: hierarquia de protocolos com percentual de frames e de bytes, destacando quais dominam o tráfego.
3.: Conectividade: retransmissões, fast retransmissions, out-of-order, zero-window, RST e falhas de TLS (alerts, versões e cipher suites incompatíveis). Informe a taxa em porcentagem, não só a contagem. Em TLS 1.3 sem chaves, o certificado não é visível: declare isso em vez de supor.
4.: Segurança: procure port scan ou IP sweep (uma origem com 20 ou mais portas ou hosts distintos com SYN sem handshake completo), força bruta (10 ou mais tentativas por minuto no mesmo serviço de autenticação), DGA e túnel DNS (domínios com alta entropia, muitos NXDOMAIN, consultas TXT ou nomes muito longos), beaconing (conexões em intervalos regulares ao mesmo destino), credenciais em texto claro e portas não padrão. Não invente reputação de IP ou domínio: sem fonte real, escreva "reputação não verificada".
5.: Latência: se algum fluxo se destacar por lentidão, compare o RTT do handshake TCP com o tempo entre a requisição e o primeiro byte da resposta, e conclua se o atraso é de rede ou de aplicação.

Antes de concluir um achado, considere se pode ser comportamento normal ou artefato do ponto de captura (checksum offload, health check, buffer cheio) e informe sua confiança (baixa, média ou alta).

Formato da resposta: comece com um veredito de 3 linhas (normal, degradado ou suspeito). Depois, uma seção por etapa, com tabelas markdown para os dados numéricos. Para cada fluxo suspeito ou problemático, mostre: IP:porta origem -> IP:porta destino | protocolo | motivo | filtro do Wireshark usado. Se uma etapa não tiver achados, escreva "Nenhuma anomalia identificada" e indique o filtro testado.

```

### Aviso de uso e privacidade
> Capturas podem conter credenciais, tokens de sessão, dados pessoais e a topologia interna da rede. Envie apenas arquivos que você está autorizado a analisar, sanitize o que não for necessário (ciente de que isso pode reduzir a precisão da análise) e confirme se a política de dados do serviço de IA é compatível com o material.
