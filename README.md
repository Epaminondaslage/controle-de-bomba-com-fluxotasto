# Controle de Bomba com Fluxostato  
## LOGO! 230R – Run-on + Anti-Ciclo + Debounce

---

## 1. Objetivo do Sistema

## Descrição do Projeto

Este projeto descreve a implementação de um sistema de controle de bomba utilizando o **PLC Siemens LOGO! 230R**, destinado a aplicações hidráulicas com **garantia de água na bomba** e que exigem **proteção contra acionamentos excessivos**. O objetivo principal é assegurar uma operação **estável, confiável** e com **maior vida útil** do conjunto eletromecânico.

A lógica de controle baseia-se em um **fluxostato**, responsável por detectar a presença de vazão e permitir o acionamento automático da bomba. Para evitar falhas causadas por oscilações rápidas do sensor, o sinal do fluxostato é tratado por um **filtro de debounce**, garantindo que apenas condições de fluxo estáveis sejam consideradas pelo sistema.

Quando ocorre perda de vazão, o sistema **não desliga a bomba imediatamente**. É aplicado um **tempo de pós-funcionamento (run-on)**, durante o qual a bomba permanece ligada por alguns segundos. Esse recurso evita desligamentos desnecessários provocados por variações momentâneas do fluxo. Após esse período, a bomba é desligada de forma controlada.

Em seguida, o sistema entra em um **modo de bloqueio temporário (anti-ciclo ou lockout)**, impedindo novas partidas por um intervalo configurável. Essa etapa protege o **motor**, o **contator** e a **instalação elétrica** contra partidas frequentes, reduzindo desgaste mecânico e térmico.

A saída do PLC aciona exclusivamente a **bobina do contator**, mantendo o circuito de potência isolado do controle. O contato do **relé térmico** é monitorado continuamente, garantindo desligamento imediato em caso de sobrecarga.

O resultado é um sistema **compacto, robusto e de fácil manutenção**, superior a soluções baseadas apenas em relés temporizadores. O projeto é indicado para aplicações **industriais, prediais e didáticas**, podendo ser facilmente expandido com **alarmes**, **contadores de partidas** ou **modos manual e automático**.


Implementa o controle de uma bomba utilizando **PLC Siemens LOGO! 230R**, atendendo aos seguintes requisitos:

1. Quando há vazão, a bomba liga normalmente  
2. Quando a vazão é perdida:
   - a bomba **continua ligada por um tempo (run-on)**
   - após esse tempo, a bomba desliga
3. Após desligar, o sistema entra em **lockout (anti-ciclo)**, impedindo religação imediata
4. Após o lockout, se a vazão estiver presente, a bomba pode ligar novamente
5. O sinal do fluxostato é tratado com **debounce (filtro)** para evitar oscilações

---

## 2. Equipamentos Utilizados

- PLC **Siemens LOGO! 230R**
- Fluxostato (contato NA – fecha com vazão)
- Relé térmico da bomba (contato NF)
- Contator da bomba (bobina 127 V)
- Fonte 127 V AC

---

## 3. Endereçamento – LOGO! 230R

| Função | Endereço |
|------|---------|
| Fluxostato (bruto) | I1 |
| Relé térmico (NF – OK = 1) | I2 |
| Fluxo filtrado (debounce) | M0 |
| Estado RUN | M1 |
| Estado RUN-ON | M2 |
| Estado LOCKOUT | M3 |
| Saída da bomba | Q1 |
| Debounce | T0 |
| Run-on | T1 |
| Lockout (anti-ciclo) | T2 |

---

## 4. Diagrama de Contatos (LADDER)

### Rede 0 – Debounce do Fluxostato
```
|----[ I1 Fluxostato ]------------------------( TON T0 300ms )----|
|----[ T0.Q ]--------------------------------------( M0 )---------|
```

### Rede 1 – Condição de RUN
```
|----[ M0 Fluxo OK ]----[ I2 Térmico OK ]----[/ M3 Lockout ]----( M1 RUN )
```

### Rede 2 – Perda de vazão (RUN-ON)
```
|----[ M1 RUN ]----[/ M0 Fluxo ]-------------------------------( M2 RUNON )
```

### Rede 3 – Temporizador Run-on
```
|----[ M2 RUNON ]--------------------------------( TON T1 5s )--|
```

### Rede 4 – Fim do Run-on → Lockout
```
|----[ T1.Q ]------------------------------------( R M1 )--------|
|----[ T1.Q ]------------------------------------( R M2 )--------|
|----[ T1.Q ]------------------------------------( S M3 )--------|
```

### Rede 5 – Temporizador Lockout
```
|----[ M3 LOCKOUT ]------------------------------( TON T2 20s )-|
```

### Rede 6 – Liberação do Lockout
```
|----[ T2.Q ]------------------------------------( R M3 )--------|
```

### Rede 7 – Comando da Bomba
```
|----[ M1 RUN ]----[ I2 Térmico OK ]----------------------------( Q1 BOMBA )
```

---

## 5. Ajustes Recomendados

- Debounce: 300 ms  
- Run-on: 5 s  
- Lockout: 20 s  

---


