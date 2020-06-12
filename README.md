# API-Graphene
graphene Django

## CSRF graphql
    https://github.com/graphql-python/graphene-django/issues/61
## condition sur les Query graphql
    https://hasura.io/docs/1.0/graphql/manual/queries/query-filters.html
## OR et AND en graphql
    https://grandstack.io/docs/graphql-filtering/
# Soit les tables suivantes et le model suivant :
    Categorie, Article, Commentaire, Tag, Comment
    
      class Categorie(models.Model):
          nom = models.CharField(max_length=250)
          date_add = models.DateTimeField(auto_now=False, auto_now_add=True)
          date_upd = models.DateTimeField(auto_now=True, auto_now_add=False)
          statut = models.BooleanField()
          def __str__(self):
              return self.nom


      class Article(models.Model):
          image = models.ImageField(upload_to='blog_image')
          titre = models.CharField(max_length=250)
          date = models.DateField(auto_now=False, auto_now_add=False)
          categorie = models.ForeignKey("Categorie", on_delete=models.CASCADE, related_name = 'CategorieArticle')
          auteur = models.ForeignKey( User , on_delete=models.CASCADE, related_name = 'ProfileArticle')
          description = models.TextField()
          content = HTMLField('Content')
          tag = models.ManyToManyField("Tag",related_name ='TagArticle')
          date_add = models.DateTimeField(auto_now=False, auto_now_add=True)
          date_upd = models.DateTimeField(auto_now=True, auto_now_add=False)
          statut = models.BooleanField()
          def __str__(self):
              return self.titre



      class Commentaire(models.Model):
          image = models.ImageField(upload_to='blog_image')
          date = models.DateTimeField(auto_now=True, auto_now_add=False)
          nom =models.ForeignKey(User, on_delete=models.CASCADE, related_name = "UserCommentaire")
          message = models.TextField()
          article = models.ForeignKey("Article", on_delete=models.CASCADE, related_name = "ArticleCommentaire")
          date_add = models.DateTimeField(auto_now=False, auto_now_add=True)
          date_upd = models.DateTimeField(auto_now=True, auto_now_add=False)
          statut = models.BooleanField()
          # def __str__(self):
          #     return self.nom


      class Tag(models.Model):
          nom = models.CharField(max_length=250)
          date_add = models.DateTimeField(auto_now=False, auto_now_add=True)
          date_upd = models.DateTimeField(auto_now=True, auto_now_add=False)
          statut = models.BooleanField()
          def __str__(self):
              return self.nom
              
      class Comment(models.Model):
          nom = models.CharField(max_length=250)
          email = models.EmailField(max_length=254)
          web = models.URLField(max_length=200)
          message = models.TextField()
          date_add = models.DateTimeField(auto_now=False, auto_now_add=True)



# Les etapes a suivre sont les suivantes :

# Etape 1 :

      pip install graphene_django
      
      
      INSTALLED_APPS = [
          ...
          'graphene_django',
      ]
     
     
# Etape 2 : creer un fichier schema.py dans l'apply

      schema.py
      
      import graphene
      from blog.models import *
      from graphene import relay, ObjectType
      from graphene_django import DjangoObjectType
      from graphene_django.filter import DjangoFilterConnectionField


      # Graphene will automatically map the Category model's fields onto the CategoryNode.
      # This is configured in the CategoryNode's Meta class (as you can see below)
      
      class CategorieNode(DjangoObjectType):
          class Meta:
              model = Categorie
              filter_fields = ['nom', 'CategorieArticle', 'date_add', 'date_upd', 'statut']
              interfaces = (relay.Node, )


      class TagNode(DjangoObjectType):
          class Meta:
              model = Tag
              filter_fields = ['nom', 'TagArticle', 'date_add', 'date_upd', 'statut']
              interfaces = (relay.Node, )

      class ArticleNode(DjangoObjectType):
          class Meta:
              model = Article
              # Allow for some more advanced filtering here
              filter_fields = {
                  'image': [],
                  'titre': ['exact', 'icontains', 'istartswith'],
                  'date': ['exact', 'icontains'],
                  'categorie': ['exact'],
                  'categorie__id': ['exact'],
                  'auteur':  ['exact'],
                  'auteur__id': ['exact'],
                  'tag': ['exact'],
                  'tag__id': ['exact'],
                  'description': ['exact', 'icontains', 'istartswith'],
                  'content': ['exact', 'icontains', 'istartswith'],
                  'date_add': ['exact', 'icontains', 'istartswith'],
                  'date_upd': ['exact', 'icontains', 'istartswith'],
                  'statut': ['exact', 'icontains', 'istartswith'],

              }
              interfaces = (relay.Node, )

      class CommentaireNode(DjangoObjectType):
          class Meta:
              model = Commentaire
              # Allow for some more advanced filtering here
              filter_fields = {
                  'image': [],
                  'nom': ['exact'],
                  'nom__id': ['exact'],
                  'date': ['exact', 'icontains'],
                  'article': ['exact'],
                  'article__id': ['exact'],
                  'message': ['exact', 'icontains', 'istartswith'],
                  'date_add': ['exact', 'icontains', 'istartswith'],
                  'date_upd': ['exact', 'icontains', 'istartswith'],
                  'statut': ['exact', 'icontains', 'istartswith'],

              }
              interfaces = (relay.Node, )


      class CommentNode(DjangoObjectType):
          class Meta:
              model = Comment
              # Allow for some more advanced filtering here
              filter_fields = {
                  'nom': ['exact', 'icontains', 'istartswith'],
                  'email': ['exact', 'icontains', 'istartswith'],
                  'web': ['exact', 'icontains', 'istartswith'],
                  'message': ['exact', 'icontains', 'istartswith'],
                  'date_add': ['exact', 'icontains', 'istartswith'],

              }
              interfaces = (relay.Node, )


      class Query(graphene.ObjectType):
          categorie = relay.Node.Field(CategorieNode)
          all_categories = DjangoFilterConnectionField(CategorieNode)

          tag = relay.Node.Field(TagNode)
          all_tags = DjangoFilterConnectionField(TagNode)

          article = relay.Node.Field(ArticleNode)
          all_articles = DjangoFilterConnectionField(ArticleNode)

          commentaire = relay.Node.Field(CommentaireNode)
          all_commentaires = DjangoFilterConnectionField(CommentaireNode)

          comment = relay.Node.Field(CommentNode)
          all_comments = DjangoFilterConnectionField(CommentNode)
     
     
     
     
     
         ######################################################## MUTATION ##################################################
     
     
     
     
     
         class RelayCreateTypeDePiece(graphene.relay.ClientIDMutation):
            type_de_chambre = graphene.Field(TypePiecesNode)
            class input:
                nom = graphene.String()
                prix = graphene.Float()
            def mutate_and_get_payload(root,info,**kwargs):
                image = info.context.FILES.get('image')
                nom = kwargs.get('nom') or None
                prix = kwargs.get('prix') or None

                if image is not None and nom is not None and prix is not None:
                    type_de_chambre = Type_pieces(image=image, nom=nom, prix=prix)
                else:
                    raise Exception('les paramettre sont requis')
                type_chambre.save()
                return RelayCreateTypeDePiece(type_de_chambre=type_de_chambre)
    
            class RelayCreateCaracteristique(graphene.relay.ClientIDMutation):
                caracteristique = graphene.Field(CaracteristiqueNode)
                class input:
                    nom = graphene.String()
                def mutate_and_get_payload(root,info,**kwargs):
                    image = info.context.FILES.get('image')
                    nom = kwargs.get('nom') or None

                    if image is not None and nom is not None:
                        caracteristique = Caracteristique(image=image, nom=nom)
                    else:
                        raise Exception('les paramettre sont requis')
                    caracteristique.save()
                    return RelayCreateTypeDePiece(caracteristique=caracteristique)

            class RelayCreateEvenement(graphene.relay.ClientIDMutation):
                evenement = graphene.Field(EvenementNode)
                class input:
                    titre = graphene.String()
                    date = graphene.types.datetime.DateTime()

                def mutate_and_get_payload(root,info,**kwargs):
                    image = info.context.FILES.get('image')
                    titre = kwargs.get('titre') or None
                    date = kwargs.get('date') or None
                    user = info.context.user or None

                    if image is not None and titre is not None and date is not None and user is not None:
                        evenement = Evenement(image=image, titre=titre, user=user, date=date)
                    else:
                        raise Exception('les paramettre sont requis')
                    evenement.save()
                    return RelayCreateEvenement(evenement=evenement)

            class RelayMutation(graphene.AbstractType):
                relay_create_type_de_pieces = RelayCreateTypeDePiece.Field()
                relay_create_caracteristique = RelayCreateCaracteristique.Field()
                relay_create_evenement = RelayCreateEvenement.Field()



     
     

# Etape 3 :

      pip install django-filter


# Etape 4 :

      GRAPHENE = {
          'SCHEMA': 'cookbook.schema.schema'
      }


# Etape 5 : creer un fichier schema.py dans le projet 

        schema.py
        
        import graphene

        import apply.schema


        class Query(apply.schema.Query, graphene.ObjectType):
            # This class will inherit from multiple Queries
            # as we begin to add more apps to our project
            pass

        class Mutation(api.schema.RelayMutation, graphene.ObjectType):
            # This class will inherit from multiple Queries
            # as we begin to add more apps to our project
            pass

        schema = graphene.Schema(query=Query, mutation=Mutation)


# Etape 6 :  dans le urls.py du projet 

      from django.conf.urls import path, include
      from django.contrib import admin

      from graphene_django.views import GraphQLView
      from projet.schema import schema


      urlpatterns = [
          path('admin/', admin.site.urls),
          path('graphql', GraphQLView.as_view(graphiql=True, schema=schema)),
      ]


