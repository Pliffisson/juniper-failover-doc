# 🔄 Failover Automático Inteligente com Juniper: Garantindo Alta Disponibilidade de Rede

*Como implementar monitoramento proativo e failover automático usando RPM Probes e Event-Options no JunOS*

---

## 🚀 O Que Este Artigo Vai Te Ensinar

Você vai aprender a criar um sistema de failover **100% automático** que:
- ✅ Monitora a conectividade em tempo real
- ✅ Detecta falhas em segundos
- ✅ Comuta rotas automaticamente
- ✅ Restaura o caminho principal quando disponível
- ✅ Não requer intervenção manual

---

## 🎯 Cenário Prático: Redundância de Links AMZ

**Situação:** Uma empresa precisa garantir conectividade constante para a rede 192.168.100.0/24 através de dois gateways:
- **Principal:** 192.168.1.1 (preferência 5)
- **Backup:** 192.168.1.2 (preferência 20)

---

## 🔍 **ETAPA 1: Configurando o Monitoramento RPM**

```bash
# Criando o probe de monitoramento
set services rpm probe TESTE_PROBE test reach-TESTE target address 192.168.1.1
set services rpm probe TESTE_PROBE test reach-TESTE probe-type icmp-ping
set services rpm probe TESTE_PROBE test reach-TESTE probe-count 5
set services rpm probe TESTE_PROBE test reach-TESTE probe-interval 3
set services rpm probe TESTE_PROBE test reach-TESTE thresholds successive-loss 5
```

### 📋 **Detalhamento Técnico:**
- **Probe Name:** `TESTE_PROBE` (identificador único)
- **Test Name:** `reach-TESTE` (nome do teste específico)
- **Target:** Gateway principal (192.168.1.1)
- **Método:** ICMP ping (mais eficiente que TCP/UDP)
- **Frequência:** A cada 3 segundos
- **Threshold:** 5 pings consecutivos perdidos = falha

---

## 🛣️ **ETAPA 2: Configurando Rotas com Preferências**

```bash
# Rota principal (preferência menor = maior prioridade)
set routing-options static route 192.168.100.0/24 qualified-next-hop 192.168.1.1 preference 5

# Rota backup (preferência maior = menor prioridade)
set routing-options static route 192.168.100.0/24 qualified-next-hop 192.168.1.2 preference 20

# Documentação da rota
set routing-options static route 192.168.100.0/24 description "TESTE - Principal e Backup com Failover"
```

### 🎯 **Como Funciona:**
- Usando IP monitoring com interface failover, você pode rastrear um endereço IP usando uma probe RPM em tempo real
- **Preference 5:** Rota sempre preferida quando ativa
- **Preference 20:** Rota usado apenas quando a principal falha
- **Qualified-next-hop:** Permite múltiplas rotas para o mesmo destino

---

## ⚙️ **ETAPA 3: Automatização com Event-Options**

### 🔴 **Policy de Falha (FAIL_TESTE_PRIMARY):**

```bash
set event-options policy FAIL_TESTE_PRIMARY events rpm
set event-options policy FAIL_TESTE_PRIMARY within 10
set event-options policy FAIL_TESTE_PRIMARY attributes match "test-name == reach-TESTE && probe-failed == true"
set event-options policy FAIL_TESTE_PRIMARY then execute-commands commands "deactivate routing-options static route 192.168.100.0/24 qualified-next-hop 192.168.1.1"
set event-options policy FAIL_TESTE_PRIMARY then commit
```

### 🟢 **Policy de Restauração (RESTORE_TESTE_PRIMARY):**

```bash
set event-options policy RESTORE_TESTE_PRIMARY events rpm
set event-options policy RESTORE_TESTE_PRIMARY within 10
set event-options policy RESTORE_TESTE_PRIMARY attributes match "test-name == reach-TESTE && probe-failed == false"
set event-options policy RESTORE_TESTE_PRIMARY then execute-commands commands "activate routing-options static route 192.168.100.0/24 qualified-next-hop 192.168.1.1"
set event-options policy RESTORE_TESTE_PRIMARY then commit
```

---

## 🔄 **Como o Failover Funciona na Prática**

### **Estado Normal (Link Principal OK):**
1. 🟢 RPM probe monitora 192.168.1.1 a cada 3 segundos
2. 🟢 Rota principal ativa (preference 5)
3. 🔴 Rota backup inativa (preference 20)
4. ✅ Tráfego flui pelo gateway principal

### **Detecção de Falha:**
1. 🔴 5 pings consecutivos falham (15 segundos)
2. ⚡ Event-policy "FAIL_TESTE_PRIMARY" é acionada
3. 🔄 Rota principal é **desativada** automaticamente
4. 🟢 Rota backup **assume** (preference 20)
5. ✅ Tráfego redirecionado em segundos!

### **Restauração Automática:**
1. 🟢 Link principal volta a funcionar
2. ✅ RPM probe detecta sucesso
3. ⚡ Event-policy "RESTORE_TESTE_PRIMARY" é acionada
4. 🔄 Rota principal é **reativada**
5. 🏠 Tráfego retorna ao caminho principal

---

## 💡 **Vantagens Desta Implementação**

### 🚀 **Performance:**
- **Detecção de falha:** ~15 segundos
- **Tempo de comutação:** < 5 segundos
- **RPO/RTO:** Praticamente zero

### 🛡️ **Confiabilidade:**
- As probes RPM não apenas testam a alcançabilidade do endereço IP, mas também realizam monitoramento de nível de serviço em parâmetros como jitter e latência
- Monitoramento contínuo
- Failback automático
- Zero intervenção manual

### 📊 **Visibilidade:**
- Logs detalhados de eventos
- Métricas de SLA em tempo real
- Histórico de performance

---

## 🔧 **Comandos de Monitoramento Essenciais**

```bash
# Verificar status das probes
show services rpm probe-results

# Monitorar eventos em tempo real
show log messages | match "rpm|event"

# Verificar rotas ativas
show route 192.168.100.0/24

# Status das event-options
show event-options policy
```

---

## ⚠️ **Boas Práticas e Considerações**

### ✅ **Recomendações:**
- **Teste regularmente:** Simule falhas para validar o funcionamento
- **Ajuste thresholds:** Adeque aos seus SLAs (successive-loss)
- **Monitor logs:** Acompanhe eventos de failover
- **Documente mudanças:** Mantenha registro das configurações

### 🚨 **Cuidados:**
- **Flapping:** Evite alternar muito rapidamente entre links
- **Testes coordenados:** Planeje manutenções para evitar failovers desnecessários
- **Backup do backup:** Considere um terceiro link para cenários críticos

---
Esta implementação oferece:
- ⚡ **Resposta rápida** a falhas de conectividade
- 🤖 **Automação completa** sem intervenção manual  
- 📈 **Alta disponibilidade** com SLA robusto
- 💰 **Custo-benefício** excelente

**Resultado:** Uma rede mais resiliente, confiável e com menos downtime!

---
**#Juniper #Networking #Failover #HighAvailability #Infrastructure #NetworkEngineering #JunOS #Automation #SLA #NetworkReliability**