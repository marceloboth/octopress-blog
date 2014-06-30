---
layout: post
title: "CRUD com Rails - Criando um cadastro usando Rspec"
date: 2014-06-28 22:22:49 -0300
comments: true
categories:
---

O acrônimo CRUD (Create, Read, Update e Delete) é algo que todo iniciante em linguagens de programação ou frameworks necessita saber, pois significa nada menos que as operações básicas para a criação/manutenção de cadastros.

O nosso primeiro passo é a parte de criação (A letra C do CRUD). Para realizar todos os passos vamos desenvolver usando as metodologias de BDD/TDD, que irá nos possibilitar um código estável e mais fácil de manter.

## Criando um cadastro

Nessa primeira história vamos criar um resource (recurso) que representa um produto, é somente para fins demonstrativos, o nome é o que menos interessa. Para o produto vamos ter dois atributos que são necessários: nome e descrição. O nosso primeiro passo é criar a nossa feature de spec, em spec/features (crie esse diretorios se não existirem), crie o diretório products e um arquivo com o nome de creating_products_spec.rb. Use sempre o inglês quando puder em sua aplicação Rails, garanto que você evitará muitas dores de cabeça.
No arquivo criado vamos especificar o que desejamos que o nosso cadastro faça. Abaixo está o código da especificação:

{% codeblock spec/features/products/creating_products_spec.rb lang:ruby %}
require 'spec_helper' # use ‘rails_helper’ caso esteja usando rails composer

feature 'Criando Produtos' do
  scenario "posso criar um produto" do
    visit '/'

    click_link 'Novo Produto'

    fill_in 'Nome', with: 'Produto 1'
    fill_in 'Descrição', with: 'Produto 1 (um)'
    click_button 'Criar produto'

    expect(page).to have_content('Produto foi criado.')
  end
end
{% endcodeblock %}

Execute o comando rspec spec/features/products/creating_products_spec.rb. Esse comando executará a nossa feature de teste e que apresentará o seguinte erro:

{% codeblock Erro no bash lang:bash %}

Failures:

  1) Criando Produtos posso criar um produto
      Failure/Error: click_link 'Novo Produto'
      Capybara::ElementNotFound:
        Unable to find link "Novo Produto"
{% endcodeblock %}

O teste falhará pois não temos o link “Novo Produto” na rota “/”, que é a nosso root (página inicial). Então antes de seguir, vamos criar o link na página inicial. Coloque o seguinte código dentro do arquivo _navigation_links.html.erb, que está na pasta layouts de nossas views (app/views/layouts/_navigation_links.html.erb):

{% codeblock app/views/layouts/_navigation_links.html.erb lang:erb %}
  <li><%= link_to 'Novo Produto', new_product_path %></li>
{% endcodeblock %}

Rode os testes novamente e ocorrera um erro pois não definimos uma rota para os nossos produtos:

{% codeblock Erro no bash lang:bash %}
  Failure/Error: visit '/'
  ActionView::Template::Error:
    undefined local variable or method 'new_product_path'
{% endcodeblock %}

Adicione a rota no arquivo config/routes.rb:

{% codeblock config/routes.rb lang:ruby %}
  resources :products
{% endcodeblock %}

Rode os testes novamente e acontecerá um nova quebra na nossa feature. Dessa vez é devido que a nossa suite de testes conseguiu encontrar a rota, mas não encontrou nenhum controller que condize-se com o resource procurado. Veja o erro:

{% codeblock Erro no bash lang:bash %}
  Failure/Error: click_link 'Novo Produto'
  ActionController::RoutingError:
    uninitialized constant ProductsController
{% endcodeblock %}

Essa ficou fácil. Crie o controller, em app/controllers/products_controller.rb. Nesse arquivo declare um classe ruby com o nome de ProductsController extendendo de ApplicationController:

{% codeblock app/controllers/products_controller.rb lang:ruby %}
class ProductsController < ApplicationController

end
{% endcodeblock %}

Rode novamente o teste. Problema resolvido, mas outro surgiu. Agora o rspec não conseguiu encontrar uma action no controller, que refletisse com a operação de novo cadastro. Veja:

{% codeblock Ação ainda não existe lang:bash %}
  Failure/Error: click_link 'Novo Produto'
  AbstractController::ActionNotFound:
    The action 'new' could not be found for ProductsController
{% endcodeblock %}

Ao controller criado instantes atrás, adicione a action new:

{% codeblock Ação de novo cadastro lang:ruby %}
class ProductsController < ApplicationController
  def new
  end
end
{% endcodeblock %}

Se rodarmos novamente os testes, nossa ação é encontrada, porém o template erb não existe, ocasionando outra quebra:

{% codeblock Erro pois o template não encontrado lang:bash %}
  Failure/Error: click_link 'Novo Produto'
  ActionView::MissingTemplate:
      Missing template products/new, application/new with ….
{% endcodeblock %}

Devemos criar o nosso template na camada de views do projeto. Então na pasta app/views, crie um pasta chamada products e nela um arquivo com o nome new.html.erb. Feito isso rode o teste mais uma vez. Agora temos um problema diferente de todos até agora. O Capybara na tentativa de preencher os campos do formulário (que não existe) acaba gerando um novo erro:

{% codeblock Não encontrou o elemento Nome lang:bash %}
  Failure/Error: fill_in 'Nome', with: 'Produto 1'
  Capybara::ElementNotFound:
       Unable to find field "Nome"
{% endcodeblock %}

Como podemos ver, o Capybara não conseguiu encontrar o elemento com o field “Nome”. Isso é lógico, nós não criamos ainda o formulário com os campos. Porém antes teremos que criar uma instância de Product em nossa controller e passar a nossa view para que ela possua os campos necessários:

{% codeblock Implementação da action new lang:ruby %}
def new
  @product = Product.new
end
{% endcodeblock %}

A constante Product  é o nosso model, que deve ser criado em app/models/product.rb. Mas antes de sair criando o arquivo, vamos acelerar as coisas usando um gerador de código do Rails.

{% codeblock Comando para gerar tudo relacionado ao model lang:bash %}
rails g model project name:string description:string
{% endcodeblock %}

Esse generator (gerador) nos poupa um tempo criando uma série de arquivos, como o arquivo de migração do banco de dados com os campos necessários, nosso model extendendo de ActiveRecord que contém um infinidade de funcionalidades para trabalhar com nossos dados e arquivos de testes unitários.

Um Model deve ser capaz de prover o acesso a camada de dados e é por isso que ele utiliza o Active Record. O Model também provê um local para a lógica de negócio, bem como, fazer validações e associações entre modelos.

Entre os arquivos gerados, está o arquivo de migration. Migration é um mecanismo eficiente de manter um controle de versão de nosso banco de dados, onde podemos evoluir e retrocer nosso schema quando necessário, sem trabalho adicional. Nosso arquivo de migração está é gerado dentro da pasta db/migrate e o nome leva em considereção o timestamp do momento de geração, para nunca ter o mesmo nome e o Rails conseguir controlar o historico de migrações. Nosso arquivo db/migrate/[data]_create_product.rb apresenta o seguite código Ruby:

{% codeblock db/migrate/[data]_create_product.rb lang:ruby %}
class CreateProducts < ActiveRecord::Migration
  def change
    create_table :products do |t|
      t.string :name
      t.string :description

      t.timestamps
    end
  end
end
{% endcodeblock %}

Devemos rodar o comando abaixo para ter o nosso banco de dados atualizado:

{% codeblock Comando para atualizar nosso banco de dados lang:bash %}
rake db:migrate
{% endcodeblock %}

O esquema de nomenclatura é interessante, o Rails irá ter o model com o nome product (singular), no entanto a tabela no banco de dados será products(plural).

Com nosso model product criado e o nosso banco com a tabela necessária, vamos voltar ao formulário. Execute o teste novamente:

{% codeblock Rodando os testes lang:bash %}
rspec spec/features/products/creating_products_spec.rb
{% endcodeblock %}

E veremos que nada mudou desde a ultima execução, porém agora temos nosso banco de dados pronto, e um objeto do tipo Product que dará os atributos necessários para montar nosso formulário de cadastro. Abra o arquivo app/views/products/new.html.erb e adicione o código abaixo:

{% codeblock app/views/products/new.html.erb lang:erb %}
<h2>New Product</h2>
<%= form_for(@product) do |f| %>
  <p>
    <%= f.label :name, "Nome" %><br />
    <%= f.text_field :name %>
  </p>
  <p>
    <%= f.label :description, "Descrição" %><br />
    <%= f.text_field :description %>
  </p>
  <p>
     <%= f.submit "Criar produto" %>
  </p>
<% end %>
{% endcodeblock %}

Formulário criado, devemos rodar nosso teste novamente. Fazendo isso veremos que nesse ponto, o que ocorre é que o formulário é preenchido, porém ao ser submitido, nenhuma ação para criar (create) é encontrado no nosso controller:

{% codeblock A ação create não esta definida lang:bash %}
Failure/Error: click_button 'Criar produto'
AbstractController::ActionNotFound:
  The action 'create' could not be found for ProductsController
{% endcodeblock %}

Então para resolver isso devemos criar o nosso método create:

{% codeblock Definindo a action create lang:ruby %}
def create
  @product = Product.new(product_params)

  if @product.save
    redirect_to @product, notice: "Produto foi criado."
  else
    flash[:alert] = "Produto não pode ser criado."

    render "new"
  end
end
{% endcodeblock %}

Também defina um  método privado para permitir que nossos parametros sejam lidos, sem serem bloqueados pelo mecanismo de Strong Parameters do Rails 4:

{% codeblock Adicionando os campos permitidos lang:ruby %}
def project_params
  params.require(:project).permit(:name, :description)
end
{% endcodeblock %}

Rode seu teste e ele ira dizer que não encontrou a action show. Isso quer dizer que o nosso cadastro está sendo salvo, porém não temos uma ação para fazer a exibição de cadastro. Adicione a ação show ao seu ProductController:

{% codeblock Nossa ação para busca e exibir o usuário lang:ruby %}
def show
  @product = Product.find(params[:id])
end
{% endcodeblock %}

Se rodarmos os testes vamos ter um erro por que o template correspondente a action, não pode ser encontrada, crie um o seguinte arquivo app/views/products/show.html.erb, e adicione o código abaixo:

{% codeblock app/views/products/show.html.erb lang:ruby %}
<h2><%= @product.name %></h2>
<p><%= @product.description %></p>
{% endcodeblock %}

Rodandos os testes, nossa feature vai passar. Se quiser testar manualmente, inicie o servidor com o comando rails s e realize um cadastro.
