# Introdução  
![Banner](https://blog.bacula.org/wp-content/uploads/2018/10/logo_bacula11.png)  
###### Dou todos os creditos dessa documentação há [Fametec](https://github.com/fametec/bacula)  
#### Bacula é um software de backup, ele é uma solução de DR (Disaster Recovery) multi plataforma.  
#### Ele tambem é o 3° software de backup mais utilizado no mundo sendo o 1º o Amanda.  
#### A documentação oficial do software pode ser obtida em: [Bacula Documentation](https://www.bacula.org/documentation/documentation/ "Clique aqui para ler a documentação")



## Director

>Este recurso define o nome do director e a senha de acesso usada para autenticação do console.  
>Apenas uma definição de recurso do Director pode aparecer no arquivo de configuração do Director.  

```
Director {                            # define myself
  Name = bacula-dir
  DIRport = 9101                # where we listen for UA connections
  QueryFile = "/opt/bacula/scripts/query.sql"
  WorkingDirectory = "/opt/bacula/working"
  PidDirectory = "/opt/bacula/working"
  Maximum Concurrent Jobs = 20
  Password = "XDnaVZYU9F4QhqUGMPxiOXsJaji23mNG3FaAM9Z2q1c/"         # Console password
  Messages = Daemon
}
```


## JobDefs

>Este recurso é opcional, fornece padrões para os recursos de Job.  

```
JobDefs {
  Name = "DefaultJob"
  Type = Backup
  Level = Incremental
  Client = bacula-fd
  FileSet = "Full Set"
  Schedule = "WeeklyCycle"
  Storage = File1
  Messages = Standard
  Pool = File
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/opt/bacula/working/%c.bsr"
}
```


## Job

>Este recurso é usado para definir os trabalhos de backup / restauração e vincular os recursos  
>Client, FileSet e Schedule a serem usados para cada trabalho. Normalmente, você criará trabalhos com nomes diferentes  
>correspondentes a cada cliente (ou seja, um trabalho por cliente, mas diferente com um nome diferente para cada cliente).  

```
Job {
  Name = "BackupClient1"
  JobDefs = "DefaultJob"
}


Job {
  Name = "BackupCatalog"
  JobDefs = "DefaultJob"
  Level = Full
  FileSet="Catalog"
  Schedule = "WeeklyCycleAfterBackup"
  # This creates an ASCII copy of the catalog
  # Arguments to make_catalog_backup.pl are:
  #  make_catalog_backup.pl <catalog-name>
  RunBeforeJob = "/opt/bacula/scripts/make_catalog_backup.pl MyCatalog"
  # This deletes the copy of the catalog
  RunAfterJob  = "/opt/bacula/scripts/delete_catalog_backup"
  Write Bootstrap = "/opt/bacula/working/%n.bsr"
  Priority = 11                   # run after main backup
}


Job {
  Name = "RestoreFiles"
  Type = Restore
  Client=bacula-fd
  Storage = File1
# The FileSet and Pool directives are not used by Restore Jobs
# but must not be removed
  FileSet="Full Set"
  Pool = File
  Messages = Standard
  Where = /tmp/bacula-restores
}
```


## FileSet

>Este recurso é usado para definir o conjunto de arquivos a serem copiados para cada Cliente.  
>Você pode ter qualquer número de FileSets, mas cada Job fará referência apenas a um.  

```
FileSet {
  Name = "Full Set"
  Include {
    Options {
      signature = MD5
    }
    File = /opt/bacula/bin
    File = /opt/bacula
  }
  Exclude {
    File = /opt/bacula/working
    File = /tmp
    File = /proc
    File = /tmp
    File = /sys
    File = /.journal
    File = /.fsck
  }
}


FileSet {
  Name = "Catalog"
  Include {
    Options {
      signature = MD5
    }
    File = "/opt/bacula/working/bacula.sql"
  }
}
```


## Schedule  

>Este recurso é usado para definir quando um trabalho deve ser executado automaticamente pelo agendador interno do Bacula.  
>Você pode ter qualquer número de agendas, mas cada Job fará referência apenas a uma.  

```
Schedule {
  Name = "WeeklyCycle"
  Run = Full 1st sun at 23:05
  Run = Differential 2nd-5th sun at 23:05
  Run = Incremental mon-sat at 23:05
}


Schedule {
  Name = "WeeklyCycleAfterBackup"
  Run = Full sun-sat at 23:10
}
```


## Client
>Este recurso é usado para definir qual cliente será feito o backup.  
>Geralmente, você terá várias definições de cliente. Cada trabalho fará referência apenas a um único cliente.  

```
Client {
  Name = bacula-fd
  Address = bacula-fd
  FDPort = 9102
  Catalog = MyCatalog
  Password = "eso80TrxzhXkRgaQVI6ZYrSzAZ4E9KFNp0Y+T1HHVWBi"          # password for FileDaemon
  File Retention = 60 days            # 60 days
  Job Retention = 6 months            # six months
  AutoPrune = yes                     # Prune expired Jobs/Files
}
```

## Autochanger ou Storage  

>Este recurso é usado para definir em qual dispositivo físico os Volumes devem ser montados.  
>As palavras-chave Storage e Autochanger são intercambiáveis. Você pode ter uma ou mais definições de Storage/Autochanger.  


```
Autochanger {  
  Name = File1  
# Do not use "localhost" here  
  Address = bacula-sd                # N.B. Use a fully qualified name here  
  SDPort = 9103  
  Password = "TS8EQJ99iLFSK39oJy33YqkZ98v4ZapjRcA+j1N6ED1n"  
  Device = FileChgr1  
  Media Type = File1  
  Maximum Concurrent Jobs = 10        # run up to 10 jobs a the same time  
  Autochanger = File1                 # point to ourself  
}  

Autochanger {  
  Name = File2  
# Do not use "localhost" here  
  Address = bacula-sd                # N.B. Use a fully qualified name here  
  SDPort = 9103  
  Password = "TS8EQJ99iLFSK39oJy33YqkZ98v4ZapjRcA+j1N6ED1n"  
  Device = FileChgr2  
  Media Type = File2  
  Autochanger = File2                 # point to ourself  
  Maximum Concurrent Jobs = 10        # run up to 10 jobs a the same time  
}  
```

## Catalog  

>Este recurso é usado para definir em qual banco de dados manter a lista de arquivos e os nomes dos volumes em que eles são copiados.  
>A maioria das pessoas usa apenas um único catálogo. No entanto, se você quiser dimensionar o Director para muitos clientes, vários catálogos  
>podem ser úteis.  
>Vários catálogos requerem um pouco mais de gerenciamento, porque geralmente você deve saber qual catálogo contém quais dados.  

```
Catalog {
  Name = MyCatalog
  dbname = "bacula"
  dbuser = "bacula"
  dbpassword = "bacula"
  DB address = "db"
}
```

## Messages  

>Este recurso é usado para definir para onde as mensagens de erro e informações devem ser enviadas ou registradas.  
>Você pode definir vários recursos de mensagens diferentes e, portanto, direcionar classes particulares de mensagens  
>para diferentes usuários ou locais (arquivos, etc.).  
```
Messages {  
  Name = Standard  
  #  mailcommand = "/opt/bacula/bin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula: %t %e of %c %l\" %r"  
  #  operatorcommand = "/opt/bacula/bin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula: Intervention needed for %j\" %r"  
  #  mail = root@localhost = all, !skipped  
  operator = root@localhost = mount  
  console = all, !skipped, !saved  
  #  append = "/opt/bacula/log/bacula.log" = all, !skipped  
  stdout = all, !skipped  
  catalog = all  
}  


Messages {  
  Name = Daemon  
  #  mailcommand = "/opt/bacula/bin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula daemon message\" %r"  
  #  mail = root@localhost = all, !skipped  
  console = all, !skipped, !saved  
  stdout = all, !skipped  
  #  append = "/opt/bacula/log/bacula.log" = all, !skipped  
}  
```


## Pool  

>Este recurso é usado para definir o conjunto de volumes que podem ser usados para um trabalho específico.  
>A maioria das pessoas usa um pool por padrão. No entanto, se você tiver um grande número de clientes ou volumes,  
>convém ter vários Pools.  
>Os pools permitem restringir um trabalho (ou um cliente) para usar apenas um conjunto específico de volumes.  
```
Pool {  
  Name = Default  
  Pool Type = Backup  
  Recycle = yes                       # Bacula can automatically recycle Volumes  
  AutoPrune = yes                     # Prune expired volumes  
  Volume Retention = 365 days         # one year  
  Maximum Volume Bytes = 50G          # Limit Volume size to something reasonable  
  Maximum Volumes = 100               # Limit number of Volumes in Pool  
}  

Pool {  
  Name = File  
  Pool Type = Backup  
  Recycle = yes                       # Bacula can automatically recycle Volumes  
  AutoPrune = yes                     # Prune expired volumes  
  Volume Retention = 365 days         # one year  
  Maximum Volume Bytes = 50G          # Limit Volume size to something reasonable  
  Maximum Volumes = 100               # Limit number of Volumes in Pool  
  Label Format = "Vol-"               # Auto label  
}  

Pool {  
  Name = Scratch  
  Pool Type = Backup  
}  
```

## Console  
>Este recurso é usado para definir qual administrador ou usuário pode usar para interagir com o Director.
```
Console {
  Name = bacula-mon
  Password = "r0V/Hx0TUwQ4TlnX1lyUHf8J8v9XvRBqnHTRW9+CB614"
  CommandACL = status, .status
}
```

## Include  
>Se desejar dividir seu arquivo de configuração em partes menores, inclua outros arquivos usando a sintaxe @caminho/completo/do/arquivo.  
>A especificação @filename pode ser fornecida em qualquer lugar em que um token primitivo apareça.  
>Se você deseja incluir todos os arquivos em um diretório específico, pode usar o seguinte:  

```
# Include subfiles associated with configuration of clients.  
# They define the bulk of the Clients, Jobs, and FileSets.  
# Remember to "reload" the Director after adding a client file.  
#@|"sh -c 'for f in /opt/bacula/etc/clientdefs/*.conf ; do echo @${f} ; done'"  

# Include bacula-dir-cloud.conf for Wasabi cloud provider   
# @/opt/bacula/etc/bacula-dir-cloud.conf  

# Include bacula-dir-cloud-aws.conf for AWS S3 cloud provider  
# @/opt/bacula/etc/bacula-dir-cloud-aws.conf  
```
