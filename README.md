# Laboratório Cloud-Init: Docker e Nginx

Solução baseada em https://github.com/carlhenriquex/cloud-init_lab e no guia de apoio fornecido. O `user-data` atualiza os índices do APT, cria `meu-usuario` com sudo sem senha e chave SSH pública, instala `docker.io` e `nginx`, escreve a página HTML e habilita os serviços. A chave privada não está incluída.

A chave incluída foi gerada no Mac do aluno. Caso o requisito de geração no Windows seja obrigatório, gere um par no Windows com `ssh-keygen -t ed25519` e substitua a linha em `ssh_authorized_keys` pelo conteúdo de `%USERPROFILE%\.ssh\id_ed25519.pub`. Nunca envie o arquivo sem `.pub`.

## Execução

O repositório original usa VirtualBox e `ubuntu/jammy64`. É necessário um provedor e box compatíveis com a arquitetura do computador; em Mac com Apple Silicon a box original pode não ser compatível. Instale uma versão compatível do VirtualBox e verifique a disponibilidade de uma box Ubuntu 22.04 ARM64 para esse provedor antes de alterar a box. Não foi verificada uma combinação compatível neste ambiente.

Na pasta deste projeto, execute:

```sh
vagrant up
vagrant reload
vagrant ssh
```

O provisionador copia os dados NoCloud e limpa o estado; o `reload` inicia o processamento. Aguarde na VM e verifique:

```sh
cloud-init status --wait --long
sudo cat /var/lib/cloud/data/result.json
id meu-usuario
sudo -n -l -U meu-usuario
sudo -u meu-usuario sudo -n id
systemctl is-enabled nginx docker
systemctl is-active nginx docker
systemctl status nginx --no-pager
systemctl status docker --no-pager
ls -l /var/www/html/index.html
sudo tail -n 80 /var/log/cloud-init-output.log
```

Confirme que o status é `done`, que não há erros no resultado e nos logs, que os dois serviços estão `active`, e que `id` lista os grupos `sudo` e `docker`. A mensagem final sozinha não comprova sucesso. No host, consulte `vagrant port` e abra `http://localhost:8080` no navegador, usando a porta efetivamente apresentada caso o Vagrant a tenha ajustado. Para testar a autenticação SSH da conta criada, consulte `vagrant ssh-config` para obter host e porta e conecte com a chave privada correspondente, mantida no computador cliente.

Salve em `evidencias/` capturas reais do terminal com o Cloud-Init, usuário e serviços, e do navegador com a página e URL. Não foram geradas capturas nesta máquina: o VirtualBox não estava instalado e a box original pode não suportar Apple Silicon.

Não inclua chaves privadas, a pasta `.vagrant` ou discos da VM. Após alterar o `user-data`, recrie a VM apenas se for seguro apagar seus dados, ou siga o procedimento apropriado de limpeza e reinicialização do Cloud-Init.

## Alternativa para Apple Silicon (ainda não validada)

O arquivo `Vagrantfile.arm64` é uma adaptação para Macs com Apple Silicon, usando a box `bento/ubuntu-22.04` ARM64. A importação da imagem foi iniciada, mas a VM ainda não foi testada até o fim: houve conflito de portas e falta de espaço. Para usá-lo, execute `VAGRANT_VAGRANTFILE=Vagrantfile.arm64 vagrant up` e siga as verificações acima. O `Vagrantfile` original foi preservado.
