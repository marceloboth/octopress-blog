---
layout: post
title: "CRUD com Rails - Carregando cadastros"
date: 2014-07-20 16:10:06 -0300
comments: true
categories: [CRUD, Rails, Iniciante]
---

Vamos começar a segunda parte, que será o carregamento dos dados. Iniciaremos mais
uma vez por nossos testes, então a primeira coisa a fazer é criar o arquivo reading_products_spec.rb
na pasta de features (``spec/features/products/reading_products_spec.rb``.

## Acessando a página
Iniciaremos nosso teste com um cenário onde estaremos navegando até a página,
então vamos ao trabalho. Implemente a navegação a página de listagem:

{% codeblock spec/features/products/creating_products_spec.rb lang:ruby %}
  require "rails_helper"

  feature "Listando Produtos" do
    scenario "listando todos os registros cadastrados" do
      visit products_path

      expect(page).to have_selector('h2', text: 'Cadastro de produtos')
    end
  end
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
      visit products_path
      expect(page).to have_selector('h2', text: 'Cadastro de produtos')
    end

    scenario "listando todos os registros cadastrados" do

    end
  end
{% endcodeblock %}

Agora vamos implementar nosso cenário de carregamento dos dados. Vamos verificar
se a nossa listagem possui ao menos um registro. Para isso vamos carregar nossa
factory e então verificar se há ao menos um registro:

{% codeblock Ao menos um registro deve existir spec/features/products/reading_products_spec.rb lang:ruby %}
  scenario "listando todos os registros cadastrados" do
    @product = FactoryGirl.create(:product)
    expect(page).to have_content(@product.name)
  end
{% endcodeblock %}

Tambem adicione na acao index do controlador o seguinte codigo, que carregara os
dados cadastrados:

{% codeblock Carregando os dados app/controllers/products_controller.rb lang:ruby %}
  class ProductsController < ApplicationController
    def index
      @products = Product.all
    end

    ... Demais actions

  end
{% endcodeblock %}

 e o nosso template index ficara assim:

{% codeblock app/views/products/index.html.erb lang:ruby %}
  <h2>Cadastro de produtos</h2>

  <table>
    <thead>
      <tr>Nome</tr>
      <tr>Descricao</tr>
    </thead>
    <tbody>
      <% if !@products.blank? %>
        <% @products.each do |product| %>
          <tr>
            <td><%= product.name %></td>
            <td><%= product.description %></td>
          </tr>
        <% end %>
      <% else %>
        Nenhum registro encontrado!
      <% end %>
    </tbody>
  </table>
{% endcodeblock %}

## Exibindo os registros
