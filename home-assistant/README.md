# Dashboard "Casa" para Home Assistant

Dashboard escuro com efeito de vidro e cores neon para **Sonoff**, **Shelly**,
**ar condicionado Wi-Fi** e **AP Ruijie**. Foi feito para se usar com um dedo:

- **tocar** num cartão liga ou desliga;
- **manter premido** abre os detalhes (brilho, histórico, definições);
- os cartões ligados acendem com a cor do tipo de equipamento: âmbar para luzes, ciano para tomadas;
- os portões ficam laranja e a piscar quando estão abertos e mostram há quanto tempo. Acionar um portão pede sempre confirmação;
- o cartão do ar condicionado muda de cor com o modo (azul a arrefecer, laranja a aquecer…)
  e a ventoinha gira mais ou menos depressa conforme a velocidade.

## Vistas

| Vista | O que tem |
|---|---|
| **Montra** | Ecrã de entrada para um tablet na parede: relógio gigante dentro de um anel de energia a brilhar, equipamentos em órbita que acendem quando estão ligados e ligados ao anel por raios de luz, fundo que muda de cor com a hora do dia, meteorologia, fechadura, clima e energia nos cantos. Quando o seu telemóvel ou o da sua mulher abre a porta, aparece "Bom dia" com o nome e a foto de quem chegou |
| **Início** | Cabeçalho animado com saudação, hora, meteorologia e resumo (temperatura, consumo, equipamentos ligados, Wi-Fi, portões) · 4 cenas rápidas · portões · luzes · tomadas e estores · ar condicionado · rede |
| **Clima** | Cada AC com termóstato completo (modos e ventilação) e gráfico de 24 h (temperatura atual e alvo) |
| **Energia** | Consumo agora, energia e custo de hoje · manómetro de potência · gráficos por equipamento |
| **Rede** | Estado do AP Ruijie (online/offline a pulsar), clientes, download/upload, tempo ativo e gráficos |

## Estrutura

```
home-assistant/
├── dashboards/casa.yaml          # o dashboard
├── themes/neon_glass.yaml        # tema
└── packages/casa_dashboard.yaml  # grupos, sensores, scripts e SNMP do Ruijie
```

## Instalação

### 1. Cartões personalizados (HACS → Frontend)

- [button-card](https://github.com/custom-cards/button-card) (obrigatório, versão 4.2 ou superior)
- [mini-graph-card](https://github.com/kalkih/mini-graph-card) (obrigatório)
- [card-mod](https://github.com/thomasloven/lovelace-card-mod) (opcional: dá o efeito de vidro aos cartões nativos)

### 2. Tema e pacote

Copie `themes/neon_glass.yaml` para `/config/themes/` e `packages/casa_dashboard.yaml`
para `/config/packages/`. Depois acrescente ao `configuration.yaml`, caso ainda não tenha:

```yaml
homeassistant:
  packages: !include_dir_named packages

frontend:
  themes: !include_dir_merge_named themes
```

Reinicie o Home Assistant. No seu **Perfil**, escolha o tema **neon_glass**.

### 3. Ligar as suas apps ao Home Assistant

Cada app do telemóvel corresponde a uma integração do Home Assistant:

| App no telemóvel | Integração no Home Assistant |
|---|---|
| **eWeLink** (Sonoff) | [SonoffLAN](https://github.com/AlexxIT/SonoffLAN), via HACS. Entre com a mesma conta eWeLink |
| **Shelly** | **Shelly**, a integração oficial. Os equipamentos aparecem sozinhos na rede local |
| **EWPE Smart** (ar condicionado Gree e marcas compatíveis) | **Gree Climate**, a integração oficial. O AC tem de estar na mesma rede |
| **Ruijie Reyee** | Não há integração. Use Ping e, se o modelo suportar, SNMP (ver as notas abaixo) |

### 4. Integrações auxiliares (pela interface)

- **Hora e data**: ative o sensor *Hora* (`sensor.time`), que atualiza o relógio do cabeçalho.
- **Ping (ICMP)**: adicione o IP do AP Ruijie e mude o ID da entidade para `binary_sensor.ruijie_ap`.
- **Meteorologia** (opcional): o cabeçalho usa `weather.forecast_casa`. Se a sua entidade tiver outro nome, altere-o em `variables.meteo` no template `cabecalho`.

### 5. Criar o dashboard

**Definições → Painéis → Adicionar painel → Novo painel do zero**. Abra-o e escolha
**⋮ → Editar → ⋮ → Editor de configuração bruta**. Cole o conteúdo de
`dashboards/casa.yaml` e grave.

## Adaptar às suas entidades

Os IDs usados são exemplos. Substitua-os pelos seus, tanto em `dashboards/casa.yaml`
como em `packages/casa_dashboard.yaml`. Uma pesquisa e substituição resolve:

| Exemplo | Equipamento |
|---|---|
| `light.sala`, `light.cozinha`, `light.quarto`, `switch.sonoff_jardim` | Sonoff (eWeLink / SonoffLAN) |
| `switch.shelly_maquina_lavar` + `sensor.shelly_maquina_lavar_power` | Shelly Plug |
| `switch.shelly_termoacumulador` + `sensor.shelly_termoacumulador_power` | Shelly 1PM |
| `cover.shelly_estore_sala`, `cover.shelly_estore_quarto` | Shelly 2PM / 2.5 em modo estore |
| `climate.ac_sala`, `climate.ac_quarto` | Ar condicionado Wi-Fi |
| `binary_sensor.ruijie_ap` | Ping ao AP Ruijie |
| `switch.shelly_portao_garagem` + `binary_sensor.portao_garagem_contacto` | Relé e sensor magnético do portão da garagem |
| `switch.sonoff_portao_rua` + `binary_sensor.portao_rua_contacto` | Relé e sensor magnético do portão da rua |

### Obter a lista das suas entidades

Em **Ferramentas de programador → Modelo**, cole o texto abaixo. Copie o resultado:
é a lista exata dos seus equipamentos e serve para substituir os nomes de exemplo
(também a pode enviar ao Claude, que faz a substituição).

```jinja
{%- for s in states if s.domain in ['light', 'switch', 'cover', 'climate', 'binary_sensor', 'weather']
     or (s.domain == 'sensor' and s.attributes.device_class in ['power', 'energy', 'temperature']) %}
{{ s.entity_id }} · {{ s.name }} · {{ s.state }}
{%- endfor %}
```

Para **acrescentar um equipamento**, copie um cartão existente e mude `entity`, `name` e `icon`:

```yaml
- type: custom:button-card
  template: luz            # ou: tomada
  entity: light.escritorio
  name: Escritório
  icon: mdi:desk-lamp
  variables:
    potencia: sensor.shelly_escritorio_power   # só nas tomadas com medição
  grid_options: { columns: 6, rows: auto }
```

Se acrescentar uma luz ou tomada, junte-a também ao grupo `casa_luzes_tomadas`. Se tiver medição de
consumo, junte o sensor de potência ao grupo `casa_sensores_potencia`.

### Ecrã Montra num tablet à entrada

- Abra o endereço do painel com `/montra` no fim, por exemplo `http://homeassistant.local:8123/painel-casa/montra`.
- Para ficar em ecrã inteiro e sempre ligado, use a app **Fully Kiosk Browser** (Android) ou o modo quiosque da app Home Assistant. Ponha o tablet sempre a carregar.
- **Boas-vindas pela fechadura Nuki Smart Lock Ultra**: quando a porta é aberta por um dos telemóveis registados, o ecrã mostra
  "Bom dia" (ou "Boa tarde" / "Boa noite"), o nome e a foto da pessoa. À noite acende também as luzes em cascata e abre os estores.
  Para configurar:
  1. Instale o add-on **Mosquitto** e a integração **MQTT** no Home Assistant.
  2. Na app Nuki: *Fechadura → Funcionalidades e configuração → MQTT*. Ative-o com o IP do Home Assistant e o utilizador do Mosquitto.
  3. Para a entidade da fechadura (`lock.porta_entrada`), use a integração **Matter** ou a descoberta automática por MQTT da Nuki.
  4. Abra a porta com cada telemóvel e veja o sensor **Nuki último evento**. O 3.º número (por exemplo `1,0,`**`123456789`**`,0,0`) é o Auth-ID desse telemóvel.
  5. No pacote, na automação `casa_boas_vindas_fechadura`, ponha esses números na lista `donos` com o nome de cada pessoa.
  6. A foto vem da entidade `person` com o mesmo primeiro nome. Defina-a em *Definições → Pessoas*.
- Os oito equipamentos em órbita são definidos em `custom_fields` (`o1` a `o8`) na vista Montra. Troque `entity`, `name`, `icon` e `cor` à vontade.

### Notas

- **Portões**: o relé que aciona o motor (Shelly 1 / Sonoff) tem de estar em modo impulso:
  *Auto off* de 1 segundo na app Shelly ou *Inching* na app eWeLink. O estado aberto/fechado vem de um
  sensor magnético. Pode ser um reed na entrada SW do Shelly ou um sensor de porta Sonoff DW2. Sem
  sensor, apague a linha `state:` do portão no pacote. O portão continua a funcionar, mas o estado
  mostrado deixa de ser fiável.

- **Consumo total**: é a soma dos sensores de potência dos Shelly. Se tiver um **Shelly EM / Pro 3EM**
  no quadro, ponha apenas esse sensor no grupo `casa_sensores_potencia`.
- **Potência contratada**: o manómetro da vista Energia vai até 6900 W. Ajuste `max:` à sua potência.
- **Preço do kWh**: defina-o na vista **Energia**. O custo de hoje é calculado a partir dele.
- **AP Ruijie**: o Home Assistant não tem uma integração oficial para a Ruijie. O pacote lê
  o tempo ativo e o tráfego por **SNMP v2c**. Ative o SNMP no AP ou no Ruijie Cloud e indique o IP e
  a community. Se o seu modelo não suportar SNMP, apague esse bloco: o estado online/offline continua
  a funcionar com o Ping e os restantes valores aparecem como "—". O número de clientes conta os
  `device_tracker` que estão em casa.
- **Cenas**: *Cheguei*, *Conforto 22°*, *Modo noite* e *Sair de casa* são scripts do pacote
  e podem ser editados à vontade. *Sair de casa* pede confirmação antes de executar.
