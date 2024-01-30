# *Unreal Product Configurator*
<img src ""Images\ .gif" width="100%"/>

## *Description*

This page is a showcase of some of the functionality I'm making at my internship at Rapid Images. Since this project is under an NDA I will be showcaseing without any context.
I'm currently uploading some of the functions I write during this project to another **github page**(https://github.com/GBaath/UnrealFunctionLibraries/tree/main/.cpp) for future use.

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
```
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
<details>
<summary>UFillGrid</summary>
```#include "FillGrid.h"

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
```
