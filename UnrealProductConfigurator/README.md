# *Unreal Product Configurator*
<img src="Images\" width="100%"/>

## *Description*

This page is a showcase of some of the functionality I'm making at my internship at Rapid Images. Since this project is under an NDA I will be showcaseing without any context.
I'm currently uploading some of the functions I write during this project to another [**github page**](https://github.com/GBaath/UnrealFunctionLibraries/tree/main/.cpp) for future use.

----

## *Current Contributions*

---

## **Optimized Preset**

This project is begin made from scratch mostly by me and another intern, and so we used the built in Unreal product configurator preset to get started.
However, that preset, from what I found is a bit cluttered, unclear, and missing scalability and flexibilty in what, most importantly and how you're locked into the preset workflow.
I've therefore begun making some modifications to most of the preset code.

## - Less dependancies and blueprint clutter

The biggest gripe I had with the default system was the setup and nesting of UMG elements. Since we weren't sure on how many configurations we were going to need I wrote new subclasses from the existing UButton and UUniformGridPanel.
Mostly so that the actual UX designing could be iterated on much faster when we recieved more info.
The changes consisted of auto adding delegates, and linking with variantset assets, as well as populating grids based on layout variables.
<details>
<summary>UVariantButton</summary>

 ```cpp

#include "VariantButton.h"

void UVariantButton::OnClickDelegate()
{
	ClickedDelegate.Broadcast(Index);
}

void UVariantButton::PostLoad()
{
	Super::PostLoad();

	OnClicked.AddUniqueDynamic(this, &UVariantButton::OnClickDelegate);
}




//TODO Make overload function with/without variantindex instead of -1 index
//negative index for the thumbnail of the whole set
void UVariantButton::NewIconFromVariantIndex(int32 SetIndex, int32 VariantIndex, ULevelVariantSets* VariantSets)
{
	UTexture2D* NewButtonTexture = nullptr;

	if (VariantIndex >= 0) 
		NewButtonTexture = VariantSets->GetVariantSet(SetIndex)->GetVariant(VariantIndex)->GetThumbnail();
	else 
		NewButtonTexture = VariantSets->GetVariantSet(SetIndex)->GetThumbnail();


	if (NewButtonTexture == nullptr) {
		//no thumbnail
		UE_LOG(LogTemp, Warning, TEXT("Missing Thumbnail"));
		return;
	}
 
	FSlateBrush Brush = GetStyle().Normal;
	Brush.SetResourceObject(NewButtonTexture);
	WidgetStyle.SetNormal(Brush);

}

```
</details>

<details>
<summary>UFillGrid</summary>

 ```cpp
#include "FillGrid.h"

void UFillGrid::UpdateContent(FFillGridData FillGridData, int32 NewContentCount, TArray<UUserWidget*>& OutArray){

	//set variables
	GridData.ContentCount = NewContentCount;
	GridData.ColumnsCount = FillGridData.ColumnsCount;
	GridData.ContainedType = FillGridData.ContainedType;


	AddAndPoolChildren();
	ManageGridLayout(OutArray);
}

void UFillGrid::AddAndPoolChildren() {

	//clear excess children
	int j = GetChildrenCount();
	for (int i = j - 1; i >= 0; i--) {
		if (i < GridData.ContentCount)
			break;
		UWidget* child = GetChildAt(i);
		if (child) {
			ChildPool.Add(child);
			child->RemoveFromParent();
		}
	}

	//re-add pooled widgets
	for (int i = j; i < GridData.ContentCount; i++) {

		if (ChildPool.Num() > 0) {
			UWidget* widget = ChildPool.Last();
			if (widget) {
				AddChild(widget);
				ChildPool.RemoveAt(ChildPool.Num() - 1);
			}
		}
		else break;
	}

	//children count could have been updated from pool
	j = GetChildrenCount();

	//Create new widgets if pool has been emptied
	for (int i = j; i < GridData.ContentCount; i++) {

		if (GridData.ContainedType == nullptr) {
			UE_LOG(LogTemp, Warning, TEXT("Missing fillgrid contain type, probably missing from the instanced struct"))
				break;
		}
		UUserWidget* widget = CreateWidget(this, GridData.ContainedType);
		AddChild(widget);
	}
}


void UFillGrid::ManageGridLayout(TArray<UUserWidget*>& OutArray) {

	OutArray.Empty();
	int i = 0;
	for (UWidget* temp : GetAllChildren())
	{
		UUniformGridSlot* Widget = UWidgetLayoutLibrary::SlotAsUniformGridSlot(temp);

		Widget->SetVerticalAlignment(VAlign_Center);
		Widget->SetHorizontalAlignment(HAlign_Center);

		//Enough children to split grid into columns?
		if (GetChildrenCount() > GridData.NrOfRowUntilColumnSplit) {
			if (GridData.ColumnsCount == 0)
				break;

			Widget->SetRow(i / GridData.ColumnsCount);
			//widgets per row
			Widget->SetColumn(i % GridData.ColumnsCount);
		}
		else{
			Widget->SetRow(i);
			//always atleast 1 column
			Widget->SetColumn(0);
		}
		//someshits wrong heeeere
		UUserWidget* casted = (UUserWidget*)temp;
		if (casted != nullptr) 
			OutArray.Add(casted);		



		i++;
	}
}

```

</details>

I also reduced a whole lot of blueprint dependencies from the preset project by nesting a lot of things that were completely unnecessarily sharing scope, actually making a night and day difference when it comes to navigating and scripting.
This is a lot nicer to work with compared to the somewhat confusing mess that is the preset product configurator preset.
[BlueprintLink](https://blueprintue.com/blueprint/2s5nlr5o/)
<img src="Images\" width="100%"/>

## - Nicer Camera Controls

The default camera was very lacking when it came to making modifications, and limiting view angles so I set out to make a better one. As I've been experimenting with a bunch of different methods of transformations, mostly getting stuck with rotators, I've managed to write a whole bunch of useful custom nodes that I stash in my [UnrealFunctionLibrary](https://github.com/GBaath/UnrealFunctionLibraries/blob/main/.cpp/CommonFunctions.cpp) repo.

The functionality that I needed for the camera were smooth stationary rotation, that could be clamped in different ways, as well as the same for camera orbiting.
In summary: I implemented these functionalities by writing a few FRotator nodes, implemented in a camera movement component (so that others can tweak the main camera settings without messing up the version control).
I found that most default rotation functions are quite confusing and hard to understand, so I kept the calculation mostly using vectors and euler angles.
The following images are a demo of my implementation of smooth claped orbiting and static rotation:
<table>
  <tr>
    <td width="50%"><img src="Images\.png" /></td>
    <td width="50%"><img src="Images\.png" /></td>
  </tr>
</table>
The custom nodes used are in a separate repo, linked above (:

## - More stuff will be added until the project is complete
