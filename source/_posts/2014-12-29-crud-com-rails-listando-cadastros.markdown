---
layout: post
title: "CRUD com Rails - Listando cadastros"
date: 2014-12-29 16:15:45 -0200
comments: true
comments: true
categories: [CRUD, Rails, Iniciante]
---

Vamos começar a segunda parte, que será o carregamento dos dados. Iniciaremos mais
uma vez por nossos testes, então a primeira coisa a fazer é criar o arquivo reading_products_spec.rb
na pasta de features (``spec/features/products/reading_products_spec.rb``.

## Acessando a página
Iniciaremos nosso teste com um cenário onde estaremos navegando até a página,
então vamos ao trabalho. Implemente a navegação a página de listagem:

{% codeblock spec/features/products/reating_products_spec.rb lang:ruby %}
  require "rails_helper"

  feature "Listando Produtos" do
    scenario "listando todos os registros cadastrados" do
      visit '/'
      click_link 'Produtos'

      expect(page).to have_selector('h2', text: 'Cadastro de produtos')
    end
  end
{% endcodeblock %}

## Organizando a navegação
Vamos adicionar um link para a pagina de listagem. No partial que contém os links
de navegação (``app/views/layouts/_navigation_links.html.erb``) substitua todo o conteúdo por:

{% codeblock app/views/layouts/_navigation_links.html.erb lang:erb %}
  <li><%= link_to 'Produtos', products_path %></li>
{% endcodeblock %}

Rode o teste com o comando ``rspec spec/features/products/reading_products_spec.rb``,
teremos um erro, pois ainda não criamos nossa ação index em nosso controlador de
produtos:

{% codeblock Erro por falta da ação index no controlador lang:bash %}
  Failures:

  1) Listando Produtos listando todos os registros cadastrados
     Failure/Error: visit products_path
     AbstractController::ActionNotFound:
       The action 'index' could not be found for ProductsController
{% endcodeblock %}

A resolução desse erro é, criar uma ação index no nosso controlador:

{% codeblock app/controller/products_controller.rb lang:ruby %}
  class ProductsController < ApplicationController
    def index

    end

    ... demais código ...

  end
{% endcodeblock %}

Rodar os testes novamente irá retornar um erro devido a falta do template para
essa ação:

{% codeblock Erro pela falta de template lang:bash %}
  Failures:

  1) Listando Produtos listando todos os registros cadastrados
     Failure/Error: visit products_path
     ActionView::MissingTemplate:
       Missing template products/index, application/index with {:locale=>[:"pt-BR"], :formats=>[:html], :variants=>[], :handlers=>[:erb, :builder, :raw, :ruby, :jbuilder, :coffee]}. Searched in:

{% endcodeblock %}

Crie o template ``app/views/products/index.html.erb``, com o conteudo:

{% codeblock Adicione o conteudo app/views/products/index.html.erb lang:ruby %}
  <h2>Cadastro de produtos</h2>
{% endcodeblock %}

Rode o teste novamente e ele irá passar.

## Carregando os dados

Na página criada vamos carregar os dados. Crie um bloco before, onde estarão os
trechos de testes que se repetirão. O before já foi utilizado no post anterior,
mas somente para lembrar.... o bloco before deve ser usado sempre que se precisa
que algo seja carregado antes dos testes, assim como é o nosso caso:

{% codeblock spec/features/products/reading_products_spec.rb lang:ruby %}
  require "rails_helper"

  feature "Listando Produtos" do

    before do
      visit root_path
    end

    scenario "listando todos os registros cadastrados" do
      click_link 'Produtos'
    end
  end
{% endcodeblock %}

Agora vamos implementar nosso cenário de carregamento dos dados. Vamos verificar
se a nossa listagem possui ao menos um registro. Para isso vamos carregar nossa
factory e então verificar se há ao menos um registro:

{% codeblock Ao menos um registro deve existir spec/features/products/reading_products_spec.rb lang:ruby %}
  scenario "listando todos os registros cadastrados" do
    @products = FactoryGirl.create_list(:product, 25)

    click_link 'Produtos'

    expect(page).to have_content(@products.first.name)
    expect(page).to have_content(@products.last.name)
  end
{% endcodeblock %}

Aqui entra o FactoryGirl, que para quem não conhece é uma gem para facilitar a criação
de fixtures e factories, evitando o uso direto do Active Record. Vamos usar para gerar
uma lista de 25 produtos, assim testando a exibição.

Primeiro vamos adiocionar a gem ao projeto, então em seu arquivo ``Gemfile``, abaixo das demais gem's do grupo
de development e test, adicione, salve e em seguida rode o comando ``bundle install``:

{% codeblock Gemfile lang:ruby %}
  group :development, :test do
    gem 'factory_girl_rails'
  end
{% endcodeblock %}

Instalada a gem, crie a factory para produtos. No diretorio ``spec`` crie uma pasta
chamada ``factories`` e nela um arquivo com nomeado ``products.rb``. Essa factory
vai mapear o modelo Product, assim tudo que é referente a manipulação de modelos
em  nossos testes, serão de responsabilidade de nossa factory. Adicione os campos
necessários para nosso teste:

{% codeblock Products spec/factories/products.rb lang:ruby %}
  FactoryGirl.define do
    factory :product do
      sequence(:name) { |n| "Produto #{n}"}
      description 'Descrição do produto 1 (um)'
    end
  end
{% endcodeblock %}

Perceba o uso do ``sequence`` na factory, ele está é usado para nunca termos nomes
iguais. Veja mais sobre o Factory Girl aqui: [site](http://www.rubydoc.info/gems/factory_girl/file/GETTING_STARTED.md)

Nosso teste se executado nesse ponto, estará quebrado, pois estamos esperando um valor
que ainda não está sendo apresentado na nossa página. Abaixo vamos implementar a exibição
dos resultados.

Adicione na acao index do controlador o seguinte código, que carregará os dados cadastrados:

{% codeblock Carregando os dados app/controllers/products_controller.rb lang:ruby %}
  class ProductsController < ApplicationController
    def index
      @products = Product.all
    end

    ... Demais actions

  end
{% endcodeblock %}

 e o nosso template index ficara assim:

{% codeblock app/views/products/index.html.erb lang:erb %}
  <h2>Cadastro de produtos</h2>

  <table class="table table-hover">
    <thead>
      <td>Nome</td>
      <td>Descricao</td>
    </thead>
    <tbody>
      <% @products.each do |product| %>
      <tr>
        <td><%= product.name %></td>
        <td><%= product.description %></td>
      </tr>
      <% end %>
    </tbody>
  </table>
{% endcodeblock %}

Rode o teste ``rspec spec/features/reading_products_spec.rb`` e nossa listagem deverá funcionar.
Caso quira inicie o server rails (``rails s``) e veja o resultado.


## Ultímos ajustes
Primeiramente rode todos os testes, para garantir que tudo está funcionando ou achar ocasionais
quebras. Execute o comando ``rspec`` e bingo, temos uma quebra. Ela ocorre devida a
modificação do nosso arquivo de navegação, onde removemos o link para o formulário de
cadastro. Para solucionar isso devemos mudar a forma como o Capybara navegava até o formulário
e principalmente definir um link na página de listagem.

Na spec de cadastro (``spec/features/creating_products_spec.rb``) altere o bloco before
para a navegação desejada:

{% codeblock spec/features/creating_products_spec.rb lang:ruby %}
  before do
    visit root_path
    click_link 'Produtos'
    click_link 'Novo Produto'
  end
{% endcodeblock %}

Ao rodar o nosso teste, ainda permanece o mesmo erro, sendo a solução adicionar esse link
em nosso template index (``app/views/products/index.html.erb``). Então a baixo da tag de
fechamento da tabela adicione o link:

{% codeblock app/views/products/index.html.erb lang:erb %}
  <h2>Cadastro de produtos</h2>

  <table class="table table-hover">
    ....
  </table>

  <%= link_to "Novo Produto", new_product_path, class: "btn btn-primary" %>
{% endcodeblock %}

Agora todos os testes estão verdes. Para finalizar vamos adicionar um teste para quando
não houver dados a serem retornados. Ao teste de carregamento adicione o cenário:

{% codeblock spec/features/reading_products_spec.rb lang:ruby %}
  scenario "exibindo mensagem de que nao ha dados" do
    click_link 'Produtos'
    expect(page).to have_content('Nenhum registro encontrado!')
  end
{% endcodeblock %}

E ao nosso template basta colocar a validação para quando a listagem estiver vazia.
Nosso template será dessa forma:

{% codeblock app/views/products/index.html.erb lang:erb %}
  <h2>Cadastro de produtos</h2>

  <div class="row">
    <% if !@products.blank? %>
    <table class="table table-hover">
      <thead>
        <td>Nome</td>
        <td>Descricao</td>
      </thead>
      <tbody>
          <% @products.each do |product| %>
            <tr>
              <td><%= product.name %></td>
              <td><%= product.description %></td>
            </tr>
          <% end %>
      </tbody>
    </table>
    <% else %>
      Nenhum registro encontrado!
    <% end %>
  </div>

  <div class="row">
    <%= link_to "Novo Produto", new_product_path, class: "btn btn-primary" %>
  </div>
{% endcodeblock %}

Agora sim chegamos ao final. Desenvolvemos o carregamento dos dados cadastrados usando
testes, com Cabybara, Rspec e FactoryGirl. No próximo post vamos ver como alterar nossos
registros e como excluí-los. O código fonte desse post está no meu [github](https://github.com/marceloboth/crud-rspec), sendo separado por branches.
