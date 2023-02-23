<h1>Template padrão para deploy do Jenkins em um  cluster AKS<h1>

![image](https://user-images.githubusercontent.com/70164638/220811843-3e9422c8-85d0-4bf4-8cf6-c4e8069178ca.png)

<h4>
Bem, como o nome do repositório já deixa claro, esses arquivos são um template para o deploy de um Jenkins em um cluster Kubernetes.

O mesmo já consta com uma service account para dar os devidos permissionamentos para o Jenkins no cluster, um service e um ingress 
para hostear nossa aplicação, um storage class e um pvc para persistencia de volume e claro, um deployment da aplicação.

Nesses arquivos existem algumas peculiaridades como por exemplo request e limit de recursos, como o Jenkins consome bastante recursos 
acabei optando por deixar até 8GB e 2CPU para o mesmo, é bastante coisa se tratando do Nó de um cluster porém na situação na qual utilizei 
esses arquivos foram limites bem tranquilos, você pode altera-los tranquilarmente para que se adeque a sua situação.

Também temos uma peculiariedade no Deployment que é o securityContext, deixei um bloco desse não comentado e um comentado no código, porém você usará 
apenas um, caso o seu Jenkins não vá utilizar nenhum volume já existente como em uma migração por exemplo, você irá utilizar o securityContext
que está comentado, assim quando ele criar os arquivos de volume do jenkins no nosso PVC os mesmos estarão linkados com o usuário do Jenkins,
caso você vá utilizar algum volume de outro lugar, após jogar os arquivos no pvc e rodar o deployment, você deverá utilizar a parte que não consta com 
comentário, pois assim ele vai rodar o container como usuário root, logo todos os arquivos de volume poderão ser iniciados normalmente, há um problema de permissionamento que ocorre caso você não utilize esse bloco de código, digamos que o seu volume no pvc não está atribuido ao usuário do jenkins mas sim a um user root ou outro, logo o jenkins vai acabar quebrando e essa foi a forma na qual achei para contornar.
<h4>

  
<h2>Utilizando os arquivos no seu cluster!<h2>
  <h3>1 --> kubectl apply -f service-account.yaml -n seu-namespace<h3>
    <h5>  O yaml da service account deve ser sempre o primeiro pois ele dará as devidas permissões para o jenkins no nosso cluster<h5>
  <h3>2 --> kubectl apply -f storage-account.yaml -n seu-namespace<h3>
    <h5>  O yaml da storage class é o carinha no qual vamos linkar o nosso pvc, assim tendo nossa perssistencia de dados garantida<h5>
  <h3>3 --> kubectl apply -f pvc.yaml -n seu-namespace<h3>
    <h5>  O yaml do pvc vai gerar o nosso pvc, (ora bolas quem diria...) com ele poderemos tanto disponibilizar arquivos de volume existentes como ter acesso aos criados pelo jenkins<h5>
  <h3>4 --> kubectl apply -f jenkinsdeployment.yaml -n seu-namespace<h3>
    <h5>  O yaml do deployment irá finalmente criar nosso pod do jenkins, caso você não tenha definido nenhum volume, o jenkins criará todos e startará normalmente, caso tenha jogado arquivos de um volume já existente no pvc, o jenkins irá ler os mesmos, carregar e iniciar<h5>
  <h3>5 --> kubectl apply -f service.yaml -n seu-namespace<h3>
    <h5 >O yaml do Service vai fazer com que tenhamos o service consumindo a porta 8080 do nosso pod do jenkins, assim consumiremos esse service no nosso ingress<h5>
  <h3>6 --> kubectl apply -f ingress.yaml -n seu-namespace<h3>
    <h5>  O yaml do ingress irá consumir o nosso service no qual aponta para a porta do nosso pod, logo supondo que teremos um dns configurado para esse cluster, ao substituir o mesmo no yaml onde fica a linha "host", a aplicação irá ser hosteada normalmente no dns.<h5>
 
<h2>   
Caso todos esses passos tenham sido seguidos e funcionado corretamente, o seu Jenkins está pronto para ser utilizado!
Quaisquer sugestões e melhorias, fiquem a vontade para issues ou pull requests!
<h2>
