# Brute Force com Kali + Medusa (Ambiente Controlado)
 Desafio — Auditoria por Força Bruta com Kali Linux e Medusa

Resumo
-----
Este projeto documenta um exercício prático de auditoria de senha por força bruta em ambiente controlado. O objetivo é aplicar conceitos vistos nas aulas usando Kali Linux e a ferramenta Medusa contra máquinas vulneráveis (Metasploitable 2 e DVWA), registrar os procedimentos, validar acessos obtidos e propor medidas de mitigação.

AVISO IMPORTANTE
-----
Este trabalho foi realizado em ambiente isolado e controlado (máquinas virtuais). Nunca execute ataques por força bruta ou outras técnicas de intrusão contra sistemas sem autorização explícita do dono/administrador da infraestrutura.

Objetivos
-----
- Compreender e demonstrar ataques de força bruta em serviços comuns: FTP, Web (formulário) e SMB.
- Configurar um ambiente de testes com VMs (Kali + Metasploitable / DVWA).
- Usar o Medusa para automatizar tentativas de senha.
- Criar e usar wordlists simples para testes.
- Documentar passos, comandos e resultados.
- Propor medidas de mitigação para as vulnerabilidades exploradas.
- Publicar evidências organizadas 

Estrutura do repositório
-----
- README.md — este arquivo.
- /wordlists — wordlists usadas nos testes.
- /images — capturas de tela dos processos e resultados..

Ambiente de Teste
-----
- VirtualBox
- 2 VMs:
  - Kali Linux (atacante)
  - Metasploitable 2 (alvo) — contém serviços vulneráveis (FTP, SMB, etc.)
  - DVWA rodando em outra VM/serviço web vulnerável para testes de formulário.
- Rede: host-only.

Preparação
-----
1. Configurar VMs:
   - Instalei Kali e Metasploitable 2 no VirtualBox.
   - Configurei ambas para usar a mesma rede interna para que se enxerguem.
2. Ferramentas:
   - Medusa (já disponível no Kali).
   - Outras ferramentas:nmap, smbclient, enum4linux.

Medidas de Mitigação e Recomendações
-----
- Políticas de senha:
  - Exigir senhas fortes (complexidade e comprimento mínimo).
  - Uso de filtros de senhas (bloquear senhas conhecidas/fracas).
- Proteção contra brute force:
  - Bloqueio temporário após X tentativas falhas (account lockout, rate limiting).
  - Implementar CAPTCHA em formulários de login.
  - Implementar throttling (limitar tentativas por IP).
- Autenticação forte:
  - Habilitar autenticação multifator (MFA).
  - Preferir autenticação baseada em chaves (em vez de senhas) quando aplicável.
- Monitoramento e detecção:
  - Logs, alertas de tentativas repetidas e análise comportamental.
  - Sistemas de prevenção (fail2ban, WAF).
- Segurança de rede:
  - Restringir exposições de serviços (acesso apenas por VPN ou rede interna).
- Patch e configuração segura:
  - Atualizar serviços e remover serviços desnecessários.
  - Garantir configurações seguras (ex.: FTP anônimo desabilitado).

