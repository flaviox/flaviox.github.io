---
layout: post
title:  "Trabalhando com componentes da paleta REST do Delphi XE7"
date:   2016-12-07
categories: delphi
tags: [delphi]
image: images/tela.png
keywords:
related:
  - title: Correios RESTful API
    url: https://correiosapi.apphb.com/
resumo: >
   Trabalhando com componentes da paleta REST do Delphi XE7 consumindo uma api utilizando método GET.
---

Neste primeiro post vou mostrar como fazer uma requisição e trabalhar com as informações retornada de uma forma bem simples. 
Como exemplo irei utilizar [link](https://correiosapi.apphb.com/){:target="_blank"} uma API que passado um CEP nos retorna o nome da rua, bairro etc.


Mãos na massa!

Vamos criar o projeto

**File > New > VCL Forms Applications - Delphi**

**1º Vamos colocar os componentes necessários na tela.**
	- TRESTClient
    - TRESTRequest
    - TRESTResponse
    - TClientDataset
    - TRESTResponseDataSetAdapter
    - TEdit (Um que iremos digitar o CEP e outros 3 para mostrar as informações)
    - TButton (Botão que iremos programar a requisição)

![Imagem]({{ page.image }})

**2º Agora vamos a configuração necessária utilizando o Object Inspector.**
	- TRESTClient
		BaseURL: http://correiosapi.apphb.com/cep/{parametro}
	
	- TRESTResponseDataSetAdapter
		Dataset: ClientDataSet1

* Os demais componentes configura de forma automática as ligações(Bindings) entre eles.

**3º Programação do botão**

```delphi
procedure TForm1.btnBuscarClick(Sender: TObject);
begin
  RESTClient1.Params[0].Value := edtCEP.Text;  //preenchemos o parametro que será submetido via metodo GET
  RESTRequest1.Execute();  //efetuamos a requisição
  if RESTResponse1.StatusCode = 200 then  // se o código da respostar for 200 ou seja tudo OK
  begin
    edtEndereco.Text := ClientDataset1.FieldByName('tipodelogradouro').AsString + ' ' + ClientDataset1.FieldByName('logradouro').AsString;
    edtBairro.Text := ClientDataset1.FieldByName('bairro').AsString;
    edtCidade.Text := ClientDataset1.FieldByName('cidade').AsString + '/' + ClientDataset1.FieldByName('estado').AsString;
    end;
end;
```

Uma outra forma seria ao invés de utilizar um Dataset, criarmos nossa propria classe com os mesmos nomes dos campos de retorno, 
```delphi
  TEndereco = class(TObject)
  private
    FLogradouro: string;
    FBairro: string;
    FCEP: string;
    FCidade: string;
    FTipoDeLogradouro: string;
    FEstado: string;
  public
    property CEP: string read FCEP write FCEP;
    property TipoDeLogradouro: string read FTipoDeLogradouro write FTipoDeLogradouro;
    property Logradouro: string read FLogradouro write FLogradouro;
    property Bairro: string read FBairro write FBairro;
    property Cidade: string read FCidade write FCidade;
    property Estado: string read FEstado write FEstado;
  end;



procedure TForm1.btnBuscarClick(Sender: TObject);
var
  Endereco: TEndereco;
begin
  Endereco := TEndereco.Create();
  try
    RESTClient1.Params[0].Value := edtCEP.Text;  //preenchemos o parametro que será submetido via metodo GET
    RESTRequest1.Execute();  //efetuamos a requisição
    if RESTResponse1.StatusCode = 200 then  // se o código da respostar for 200 ou seja tudo OK
    begin
      Endereco := TJson.JsonToObject<TEndereco>(RESTResponse1.Content);  //Metodo se utiliza da RTTI para preencher nossa classe
      edtEndereco.Text := Endereco.TipoDeLogradouro + ' ' + Endereco.Logradouro;
      edtBairro.Text := Endereco.Bairro;
      edtCidade.Text := Endereco.Cidade + '/' + Endereco.Estado;
    end;
  finally
    Endereco.Free();
  end;
end;


```




Gostou? Deixe abaixo seu comentário!


