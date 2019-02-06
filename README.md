# vagrant-barman
Vagrantfile para montar um ambiente de backup com PostgreSQL 10 e Barman 2.6

Para subir o ambiente:
  vagrant up
  
Para testar o bakcup primeiro logue na maquina com o barman
  vagrant ssh barman

Apos logar teste a conexão com o banco
  barman list-server
  barman check pg
  
Inicie o Backup
  barman switch-wal pg
  barman check pg
  barman backup pg
  barman list-backup pg
