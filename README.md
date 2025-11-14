# Configuração dos Daemons Hyper-V no Ubuntu

### (KVP, VSS e FCopy totalmente funcionais + Correção dos erros de DNS/DHCP)

Este guia fornece o passo a passo completo para habilitar e corrigir os serviços Hyper-V no Ubuntu:

- **KVP (Key-Value Pair Daemon)**
- **FCopy (File Copy Daemon)**
- **VSS (Volume Shadow Copy Daemon)**

Compatível com Hyper-V Standalone, Failover Cluster e Azure Stack HCI.

---

## 1. Verificar a versão atual do kernel

Os pacotes Hyper-V precisam corresponder *exatamente* ao kernel carregado.

```bash
uname -r
```

Exemplo de saída:
```
5.15.0-157-generic
```

---

## 2. Atualizar o sistema

```bash
sudo apt update
sudo apt upgrade -y
```

---

## 3. Instalar ferramentas do kernel correspondente

```bash
sudo apt install linux-tools-$(uname -r) linux-cloud-tools-$(uname -r) -y
```

### Instalar as versões genéricas (recomendado):

```bash
sudo apt install linux-tools-generic linux-cloud-tools-generic -y
```

Os pacotes instalam:
- `hv_kvp_daemon`
- `hv_fcopy_daemon` (Não é necessário se seu objetivo for integração Hyper-V)
- `hv_vss_daemon`

---

## 4. Limpar Módulo Hyper-V

Limpe o módulo Hyper-V

```bash
sudo modprobe -r hv_utils
```

---

## 5. Reiniciar os serviços e habilitar módulo e daemons

```bash
sudo modprobe hv_utils
sudo systemctl restart hv-kvp-daemon.service
sudo systemctl restart hv-fcopy-daemon.service
sudo systemctl restart hv-vss-daemon.service
sudo systemctl enable hv-kvp-daemon.service
sudo systemctl enable hv-fcopy-daemon.service
sudo systemctl enable hv-vss-daemon.service
```

---

## 6. Verificar se tudo foi instalado corretamente

```bash
systemctl status hv-kvp-daemon.service
systemctl status hv-fcopy-daemon.service
systemctl status hv-vss-daemon.service
```

Ou verificar todos de uma vez:

```bash
systemctl list-units --all | grep hv-
```

---

## Resolução de Problemas

### Daemon não inicia

Se algum daemon não iniciar, verifique os logs:

```bash
sudo journalctl -u hv-kvp-daemon.service -n 50
sudo journalctl -u hv-fcopy-daemon.service -n 50
sudo journalctl -u hv-vss-daemon.service -n 50
```

### Módulos não carregam

Verifique se os módulos do kernel estão disponíveis:

```bash
modinfo hv_utils
modinfo hv_vmbus
```

### Reinstalar os pacotes

Se necessário, reinstale os pacotes:

```bash
sudo apt remove --purge linux-tools-$(uname -r) linux-cloud-tools-$(uname -r)
sudo apt autoremove
sudo apt install linux-tools-$(uname -r) linux-cloud-tools-$(uname -r) -y
```

---

## Notas Importantes

1. **Kernel atualizado**: Após cada atualização do kernel, pode ser necessário reinstalar os pacotes `linux-tools` e `linux-cloud-tools` correspondentes.

2. **Compatibilidade**: Este guia é compatível com Ubuntu 18.04, 20.04, 22.04 e 24.04 LTS.

3. **Permissões**: Todos os comandos requerem privilégios de superusuário (sudo).

4. **Integração completa**: Com os três daemons funcionando, você terá integração completa com Hyper-V, incluindo:
   - Troca de dados entre host e guest (KVP)
   - Cópia de arquivos via Hyper-V Manager (FCopy)
   - Backup consistente da VM (VSS)

---

## Verificação Final

Para confirmar que tudo está funcionando:

1. No host Hyper-V, abra o Hyper-V Manager
2. Selecione a VM Ubuntu
3. Verifique o status dos Integration Services
4. Todos os serviços devem estar com status "OK" ou "Enabled"

---

**Guia criado para integração completa entre Ubuntu e Hyper-V**
