---
layout: post
title: "De la 2D avec Unity"
published: true
categories: gamedev
---

__Update__ Depuis la version 4.3 la création de jeu vidéo 2D est incluse dans Unity : [2D tutorials](http://unity3d.com/learn/tutorials/modules/beginner/2d).

Même si Unity 3D est moteur de jeu 3D, il reste toujours un moteur intéressant pour faire de la 2D. Dans cet article je vais montrer les techniques pour faire un jeu 2D avec des sprites. 

A l'origine c'est l'article [Making 2D Games With Unity](http://gamasutra.com/blogs/JoshSutphin/20130519/192539/Making_2D_Games_With_Unity.php) qui m'a motivé pour faire de la 2D avec Unity. Les explications que je vais donner sont assez légères, pour me rattrapper je vous donne tout les liens qui m'ont aidé :)

L'éditeur d'Unity est puissant, mais comme je suis un développeur et que je débute sur Unity, je préfère faire le maximum possible avec des scripts C#. 

Quand on crée un projet Unity, on se retrouve avec une scène vide et une caméra. La première chose à faire est de créer un empty `GameObject`. On va ensuite y attacher un script `MainScript` qui sera notre "main". Pour en savoir plus sur le scripting avec Unity, vous pouvez lire la doc sur le site officiel ou visiter ce [site](http://3dgep.com/?p=3474).

Le `MainScript` généré par Unity : 

{% highlight c# %}
using UnityEngine;

public class MainScript : MonoBehaviour {
	void Start () {}
	void Update () {}
}
{% endhighlight %} 

La méthode `Start` est appelée une fois lorsque les GameObjects de la scène ont été crées, c'est à ce moment la que l'on va faire le travail d'initialisation notamment la création de la caméra.

{% highlight c# %}
using UnityEngine; 

public class MainScript : MonoBehaviour {
	private Camera _cam;
	
	void Start () {
		this.SetupCamera();
	}
	
	void Update () {}
}
{% endhighlight %}

Notez que j'ai crée un attribut `_cam` pour stocker la référence de notre future caméra.

Unity ne supporte pas la création d'une caméra a partir de rien, alors on va garder la caméra par défaut dans la scène pour servir de modèle.

La méthode `SetupCamera` qui suit est issue de l'article [Creating a camera using C# in Unity](http://shadowmint.blogspot.fr/2012/10/creating-camera-using-c-in-unity.html).

{% highlight c# %}
// Setup an orthographic camera at (0, 0, -1) aiming at (0, 0, 1)
// The vertical half-size of the orthographic camera is 3 units.
public void SetupCamera() {
	var original = GameObject.FindWithTag("MainCamera");
	_cam = (Camera) Camera.Instantiate(
		original.camera,
		new Vector3(0, 0, -1),
		Quaternion.FromToRotation(
			new Vector3(0, 0, 0),
			new Vector3(0, 0, 1)
		)
	);
	_cam.orthographicSize = 3;
	_cam.orthographic = true;
	_cam.backgroundColor = Color.green;
	_cam.depth = 0;
	_cam.enabled = true;
	GameObject.Destroy(original);
}
{% endhighlight %}

On a une caméra ortographique, c'est à dire que les objets n'apparaissent pas plus petit lorsque leur distance à la caméra augmente. L'utilisation de la projection ortographique est typique du jeu en 2D. 

La caméra est orientée de façon à faire façe au Z positif, cela permet de faire correspondre l'axe X et Y de la scène a celui de notre affichage 2D. 

Maintenant que la caméra est prête, on va travailler sur les sprites. On va créer un fichier `Sprite.cs` qui va contenir la classe `Sprite`. 

Il nous faut une structure plane qui va être texturé avec les futures images du jeu, mais on ne va pas utiliser le `Plane` de Unity car c'est une grille de 10x10 quads. On peut économiser des ressources en créant notre propre `Mesh` constitué d'un seul quad. Ici j'ai suivi l'article [Display a textured 2D quad in Unity using code only](http://shadowmint.blogspot.fr/2012/11/display-textured-2d-quad-in-unity-using.html). 

{% highlight c# %}
using UnityEngine; 
public class Sprite {
	private static Mesh CreateMesh() {
		Mesh mesh = new Mesh();
		
		Vector3[] vertices = new Vector3[] {
			new Vector3(0, 0, 0),
			new Vector3(1, 0, 0),
			new Vector3(0, 1, 0),
			new Vector3(1, 1, 0),
        };
		
		Vector2[] uv = new Vector2[] {
			new Vector2(0, 0),
			new Vector2(1, 0),
			new Vector2(0, 1),
			new Vector2(1, 1),
		};
		
		int[] triangles = new int[] {
			2,3,1, 1,0,2
		};
		
		mesh.vertices = vertices;
		mesh.uv = uv;
		mesh.triangles = triangles;
		mesh.RecalculateNormals();
		
		return mesh;
	}
}
{% endhighlight %} 

Pour plus de détails sur la création du `Mesh`, vous pouvez lire la page [custom_meshes](http://hub.jmonkeyengine.org/wiki/doku.php/jme3:advanced:custom_meshes) du wiki de JMonkeyEngine. 

On a crée un `Mesh` qui face le Z négatif, c'est à dire que sa face texturée est tournée vers la caméra pour qu'il soit visible. 

Dans Unity, il faut attribuer un `Material` aux objets pour indiquer comment ils vont être rendus. Une des propriétés du `Material` est l'image qui servira de texture. Dans le jeu on va avoir des textures différentes pour chaque élément du jeu, mais on ne va pas créer plusieurs materials car chaque changement de `Material` lors du rendu requiert des ressources GPU et CPU. On va créer un seul `Material` dont la texture sera un sprite atlas (une image qui regroupe tout les sprites du jeu). Cette technique permet de bénéficier du [draw call batching](http://docs.unity3d.com/Documentation/Manual/DrawCallBatching.html). 

Voici un exemple de sprite atlas :

![atlas]({{ site.url }}/assets/terrain.png)

{% highlight c# %}
private static Material CreateAtlas() {
	Material mat = new Material(Shader.Find ("Unlit/Transparent"));
	
	// Set texture
	var tex = (Texture) Resources.Load("atlas");
	mat.mainTexture = tex;
	
	return mat;
}
{% endhighlight %}

Le fichier `atlas.png` doit être mis dans un fichier `Resources` lui même situé n'importe où dans `Assets`. Quand on place l'image dans les dossier, elle apparaît dans les Assets d'Unity. Avec l'inspector, on effectue les réglages suivant :

	Filter Mode : point
	Max Size : 1024
	Format : Truecolor 

Comme tout les sprites vont avoir la même texture, on utilise les coordonnées UV pour positionner la bonne partie de l'image sur les triangles. Dans la méthode suivante, je crée les UVs en fonction des coordonnées du sprite dans l'atlas et j'en profite pour scaler le quad selon la forme et la taille du sprite.

{% highlight c# %}
public void SetTexture(int px, int py, int pw, int ph) {
	// setting the material
	this.gameObject.renderer.material = Sprite.atlas;
	
	// setting the mesh
	this.gameObject.GetComponent<MeshFilter>().mesh = Sprite.mesh;
	
	// scaling
	float scaleX = (float) pw / (float) Sprite.minSize;
	float scaleY = (float) ph / (float) Sprite.minSize;
	this.gameObject.transform.localScale = new Vector3(scaleX, scaleY, 0);
	
	// setting UVs
	Mesh mesh = this.gameObject.GetComponent<MeshFilter>().mesh;
	Vector2[] uvs = new Vector2[mesh.uv.Length];
	Texture texture = Sprite.atlas.mainTexture;
	
	Vector2 pixelMin = new Vector2(
		(float) px / (float) texture.width,
		1.0f - ((float) py / (float)texture.height)
	);
	
	Vector2 pixelDims = new Vector2(
		(float) pw / (float) texture.width,
		-((float) ph / (float)texture.height)
	);
	
	Vector2 min = pixelMin;
	uvs[0] = min + new Vector2(pixelDims.x * 0.0f, pixelDims.y * 1.0f);
	uvs[1] = min + new Vector2(pixelDims.x * 1.0f, pixelDims.y * 1.0f);
	uvs[2] = min + new Vector2(pixelDims.x * 0.0f, pixelDims.y * 0.0f);
	uvs[3] = min + new Vector2(pixelDims.x * 1.0f, pixelDims.y * 0.0f);
	mesh.uv = uvs;
}
{% endhighlight %}

`px` et `py` sont les coordonnées du coin haut gauche du sprite dans l'atlas. `pw` et `ph` sont respectivement la largeur et la hauteur du sprite.

Notez les attributs statiques qui ne sont pas encore définis. On a assez d'éléments pour finir la classe `Sprite`. Voila le code complet : 

{% gist 6225954 %}

On peut maintenant créer des Sprites dans la méthode start de notre main : 

{% highlight c# %}
void Start () {
	SetupCamera();
	
	Sprite s1 = new Sprite();
	s1.SetPosition(-7, 0, 0);
	s1.SetTexture(0, 0, 32, 8);
	Sprite s2 = new Sprite();
	s2.SetPosition(-2, 0, 0);
	s2.SetTexture(0, 0, 16, 16);
	Sprite s3 = new Sprite();
	s3.SetPosition(2, -2, 0);
	s3.SetTexture(0, 0, 16, 64);
}

{% endhighlight %}

Note : Pour que le jeu fonctionne en dehors de l'éditeur, il faut que la scène contienne un objet qui a le shader utilisé sur notre `Material`.