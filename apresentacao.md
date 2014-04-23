# Gerenciando o WordPress com o WP-CLI

Rodrigo Primo

WordCamp São Paulo 2013

.fx: titleslide

---

# Quem sou eu

* Sócio e desenvolvedor no Hacklab
* Trabalho com WordPress desde 2009
* Defensor do software livre

---

# O que é o WP-CLI

* É um conjunto de ferramentas para gerenciar o WordPress a partir da linha de comando.
* Permite atualizar plugins, alterar opções, instalar temas, entre outras funções.
* Escrito em PHP e publicado como software livre (MIT Public license).
* Construído reaproveitando o código do próprio WP.
* Criado por Cristi Burcă (scribu) e Andreas Creten em 2011.

---

# Para que serve?

---

## Automação

Backups periódicos da base de dados:

    !shell-session
    #!/bin/bash

    cd /diretorio/wp/

    wp db export /diretorio/backup/wordpress.sql

---

## Integração

Script Python para pegar informações de um usuário:

    !python
    import subprocess
    
    def get_wp_user_info(user_name):
        command = ["wp", "user", "get", user_name]
    
        proc = subprocess.Popen(command,
            stdout=subprocess.PIPE)
        out, err = proc.communicate()
    
        return out.strip("\n")
    
    print get_wp_user_info("admin")
   
---

## Desenvolvimento

Geração de código:

    !shell-session
    $ wp scaffold plugin wordcamp
    $ wp scaffold post-type slides --plugin=wordcamp
    
Depuração:

    !shell-session
    $ wp shell
    wp> get_bloginfo('blogname')
    string(13) "WordCamp"
    wp> get_the_title(1)
    string(23) "WordCamp São Paulo 2013"

---

## Administração

Atualizar o WP, plugins e temas:

    !shell-session
    $ wp core update
    $ wp core update-db
    $ wp theme update --all
    $ wp plugin update --all

---

# Requisitos

* Acesso ao shell
* Linux ou OS X
    * Suporte não oficial ao Windows através do projeto WP-PowerShell - https://github.com/ericmann/WP-PowerShell
* PHP >= 5.3.2 e php5-cli
* WordPress >= 3.4

---

# Como instalar

No terminal:

    !shell-session
    $ curl https://raw.github.com/wp-cli/wp-cli.github.com/master/
    installer.sh | bash

Adicionar as linhas abaixo ao .bash_profile para auto-complete dos comandos:

    !shell-session
    # WP-CLI Bash completions
    source ~/.wp-cli/vendor/wp-cli/wp-cli/utils/wp-completion.bash

Mais informações em http://wp-cli.org

---

# Uso básico

Lista dos comandos disponíveis:

    !shell-session
    $ wp

Ajuda de um comando específico:

    !shell-session
    $ wp help theme

Ajuda de um sub-comando específico:

    !shell-session
    $ wp help theme list

Estrutura dos comandos:

    !shell-session
    $ wp comando subcomando --assoc_arg1=value1 arg1
    $ wp user get --format=json admin

---

# Mais exemplos de comandos

---

## Baixar e instalar o WP

Baixar:

    !shell-session
    $ wp core download

Criar o wp-config.php:

    !shell-session
    $ wp core config --dbname=baseDeDados --dbuser=usuario

Instalar:

    !shell-session
    $ wp core install --url=http://wc.dev --title=WC
    --admin_user=admin --admin_password=wc
    --admin_email=wordcamp@wordpress.org

---

## Tarefas de administração

Instalar plugin:

    !shell-session
    $ wp plugin install debug-bar

Instalar tema:

    !shell-session
    $ wp theme install p2

Ativar tema:

    !shell-session
    $ wp theme activate twentytwelve

---

## Tarefas de desenvolvimento

Alterar uma string no banco de dados (em especial a URL do WP):

    !shell-session
    $ wp search-replace textoAntigo textoNovo

Ver o valor de uma opção serializada:

    !shell-session
    $ wp option get sidebars_widgets

Gerar dados para teste:

    !shell-session
    $ wp post generate --count=500

---

## Banco de dados

Exporta a base de dados para um arquivo SQL:

    !shell-session
    $ wp db export dump.sql

Importa um arquivo SQL para a base de dados:

    !shell-session
    $ wp db import dump.sql

Roda uma query no banco:

    !shell-session
    $ wp db query "select * from wp_users"

---

## Combinando comandos

Deletar um conjunto de posts:

    !shell-session
    $ wp post delete $(wp post list --post_type='post' --format=ids)

---

# O WP-CLI é extensível

* Existe uma API que permite a criação de novos comandos ou subcomandos.
* Um novo comando pode ser distribuído através de um plugin ou pode ser incluído localmente no arquivo de configuração do WP-CLI.
* Para mais informações sobre como criar um comando: https://github.com/wp-cli/wp-cli/wiki/Commands-Cookbook
* Lista de comandos criados pela comunidade disponível em: https://github.com/wp-cli/wp-cli/wiki/List-of-community-commands

---

# CLI para o wp-super-cache

Carregar novos comandos:

    !php
    function wpsc_cli_init() {
        if ( !function_exists( 'wp_super_cache_enable' ) )
            return;
    
        if ( defined('WP_CLI') && WP_CLI ) {
            include dirname(__FILE__) . '/cli.php';
        }
    }
    
    add_action( 'plugins_loaded', 'wpsc_cli_init' );

Fonte: https://github.com/wp-cli/wp-super-cache-cli

---

# CLI para o wp-super-cache

Adicionar novo comando no arquivo cli.php:

    !php
    WP_CLI::add_command( 'super-cache', 'WPSuperCache_Command' );
    
    /**
     * Command line interface for wp-super-cache
     */
    class WPSuperCache_Command extends WP_CLI_Command {
        [...]
    }
    
---

# CLI para o wp-super-cache

Exemplo de comando:

    !php
    /**
     * Clear something from the cache.
     *
     * @synopsis [--post_id=<post-id>]
     */
    function flush( $args = array(), $assoc_args = array() ) {
        [...]
    }

---

# CLI para o wp-super-cache

    !php
    function flush( $args = array(), $assoc_args = array() ) {
        if ( isset($assoc_args['post_id']) ) {
            if ( is_numeric( $assoc_args['post_id'] ) ) {
                wp_cache_post_change( $assoc_args['post_id'] );
                WP_CLI::success( 'Cache cleared.' );
            } else {
                WP_CLI::error( 'This is not a valid post id.' );
            }
            wp_cache_post_change( $assoc_args['post_id'] );
        } else {
            global $file_prefix;

            wp_cache_clean_cache( $file_prefix, true );
            WP_CLI::success( 'Cache cleared.' );
        }
    }

---

# Como contribuir

* http://wp-cli.org
* &#35;wordpress-cli no irc.freenode.net
* https://github.com/wp-cli/wp-cli/issues

---

# Obrigado!

__Rodrigo Primo__

rodrigo@hacklab.com.br

http://rodrigoprimo.com

http://github.com/rodrigoprimo

.fx: lastslide  

