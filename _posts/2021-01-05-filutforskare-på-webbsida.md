---
layout: post
title: Filutforskare på webbsida
categories: programmering
tags: asp.net C# javascript
---

# Filutforskare på webbsida

Jag ville ha en filutforskare på en webbsida som inte innebar att användaren ska klicka runt i en trädstruktur och byggde då en lösning som liknar den i google drive. Här beskriver jag hur jag gjorde.

![Filutforskare på webbsida](/assets/img/filutforskare-webb.png)

Obs! Den här implementationen är inte gjord för stora mängder filer eftersom den laddar in alla då sidan öppnas, men funkar utmärkt med ett hundratal filer. Fördelen är att den är väldig snabb eftersom den inte behöver ladda något från servern när man bläddrar mellan foldrar.  

Filerna ligger i en platt struktur i en egen "store" med metadata i en databas.

Så här ser entiteten ut:

{% highlight csharp %}
public class FileModel 
{
	public int Id { get; set; }
	public string Folder { get; set; }
	public string Title { get; set; }
	public string Filename { get; set; }
}
{% endhighlight %}

Vi hämtar ut en lista med entiter och omvandlar dem till ett träd baserat på Folder.

{% highlight csharp %}
public static FileTree ListToTree(List<FileModel> items)
{
	FileTree root = new FileTree();

	foreach (var item in items)
	{
		FileTree currentNode = root;

		var folders = item.Folder.Split(new char[] { '/' }, StringSplitOptions.RemoveEmptyEntries);
		foreach (var folder in folders)
		{
			currentNode = currentNode.AddChild(folder);
		}
		currentNode.AddChild(item);
	}

	return root;
}
{% endhighlight %}

Så här ser FileTree ut:

{% highlight csharp %}
public class FileTree
{
	public string Name { get; private set; }
	public FileModel File { get; private set; }
	public List<FileTree> Children { get; private set; } = new List<FileTree>();
	public string Path { get; private set; } = string.Empty;
	public bool IsRoot { get; private set; } = false;

	public FileTree()
	{
		IsRoot = true;
		Path = Name = "Dokument";
	}

	private FileTree(string name)
	{
		Path = Name = name;
	}

	private FileTree(FileModel data)
	{
		this.Name = data.Title;
		this.File = data;
	}

	public FileTree AddChild(string name)
	{
		var item = Children.Count > 0 ? Children.Find(p => p.Name.Equals(name)) : null;
		if (item != null) return item;

		var newNode = new FileTree(name);
		newNode.Path = string.Join("_", Path, name); 
		this.Children.Add(newNode);
		return newNode;
	}

	public FileTree AddChild(FileModel data)
	{
		FileTree newNode = new FileTree(data);
		newNode.Path = string.Join("_", Path, data.Filename); 
		this.Children.Add(newNode);
		return newNode;
	}

	public bool IsFolder { get { return File == null; } }
}
{% endhighlight %}


Vi använder en partiell vy rekursivt för att generera paneler med innehållet för varje folder:

{% highlight html %}
@model FileTree

@{
	var node = ViewData.Model;

	// render panel
	<div class="panel" style="@(node.IsRoot ? "display: block;" : "")" id="tab_@node.Path">
		<table class="table">
			<thead>
				<tr><td>Namn</td></tr>
			</thead>
			<tbody>
				@foreach (var item in node.Children)
				{
					<tr><td>
						@if (item.IsFolder)
						{
							<img src="~/Content/Images/Icons/folder.svg" />
							<a class="panel-folder-link" id="link_@item.Path" href="#tab_@item.Path">@item.Name</a>
						}
						else
						{
							<img src="@FileHelper.GetIconPath(item.File.Filename)" />
							<a class="panel-file-link" id="link_@item.Path" href="@item.File.GetLink()">@item.Name</a>
						}
					</td></tr>
				}
			</tbody>
		</table>
	</div>

	// iterate over children
	foreach (var item in node.Children)
	{
		if (item.IsFolder)
		{
			Html.RenderPartial("DocumentItemControl", item);
		}
	}
}
{% endhighlight %}

Huvudvyn ser ut så här:

{% highlight html %}
@{
	ViewBag.Title = "Dokument";
}

@section MainContent
{
	<h2>Dokument</h2>

	<br />

	<nav style="--bs-breadcrumb-divider: '>';" aria-label="breadcrumb">
		<ol class="breadcrumb" id="breadcrumb">
			<li class="breadcrumb-item active">Dokument</li>
		</ol>
	</nav>

	<div class="tab-content" id="nav-tabContent">
		@{ Html.RenderPartial("DocumentItemControl", ViewData["root"]); }
	</div>

	<br />
	<br />
}
{% endhighlight %}

Och här är javascript för att växla vilken panel som är synlig samt för att uppdatera breadcrumb.

{% highlight javascript %}
$(function () {
	$(document).on('click', '.panel-folder-link', function (evt) {
		var showPanelId = this.id.replace(/\blink_/, "");
		evt.preventDefault();
		switchVisiblePanel('tab_' + showPanelId);
		updateBreadcrumb(showPanelId);
	});

	function switchVisiblePanel(targetPanelId) {
		$('.panel').hide();
		document.getElementById(targetPanelId).style.display = "block";
	}

	function updateBreadcrumb(targetPanelId) {
		var parts = targetPanelId.split("_");
		var html = "";
		var linkId = "link";
		parts.forEach(function (item) {
			if (item === parts[parts.length - 1]) {
				html += '<li class="breadcrumb-item active">' + item + "</li>";
			}
			else {
				linkId += "_" + item;
				html += '<li class="breadcrumb-item"><a class="panel-folder-link" id="' + linkId + '" href="#">' + item + "</a></li>";
			}
		});
		document.getElementById("breadcrumb").innerHTML = html;
	}
});
{% endhighlight %}

Kanske inte den snyggaste koden, men det funkar.
