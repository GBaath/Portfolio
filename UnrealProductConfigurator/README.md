# *Unreal Product Configurator*

## *Description*

This page is a showcase of some of the functionality I'm making at my internship at Rapid Images. Since this project is under an NDA I will be showcaseing without any context.
I'm currently uploading some of the functions I write during this project to another [**github page**](https://github.com/GBaath/UnrealFunctionLibraries/tree/main/.cpp) for future use.

----

## *Current Contributions*

---

## **Optimized Preset**

This project is made from scratch mostly by me and another intern, and so we used the built in Unreal product configurator preset to get started.
However, that preset, from what I found is a bit cluttered, unclear, and missing scalability and flexibilty in what, most importantly and how you're locked into the preset workflow.
I've therefore begun making some modifications to most of the preset code.

## - Less dependancies and blueprint clutter

The biggest gripe I had with the default system was the setup and nesting of UMG elements. Since we weren't sure on how many configurations we were going to need I wrote new subclasses from the existing UButton and UUniformGridPanel.
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

I also reduced a whole lot of blueprint dependencies from the preset project by nesting a lot of things that were completely unnecessarily sharing scope, making a night and day difference when it comes to navigating and scripting.
[BlueprintLink](https://blueprintue.com/blueprint/2s5nlr5o/)

<img src="Images\VManagerControls_demo.png" width="50%"/>

## - Nicer Camera Controls

The default camera was very lacking when it came to making modifications, and limiting view angles so I set out to make a better one. 
I've managed to write a whole bunch of useful custom nodes that I stash in my [UnrealFunctionLibrary](https://github.com/GBaath/UnrealFunctionLibraries/blob/main/.cpp/CommonFunctions.cpp) repo.

The functionality that I needed for the camera were smooth stationary rotation, that could be clamped in different ways, as well as the same for camera orbiting.
In summary: I implemented these functionalities by writing a few FRotator nodes, implemented in a camera movement component.
During the development of this functionality I've developed a much better understanding of, and now feel very comfortable working with quaternions and more advanced geometric algebra.
[Orbit Demo](https://blueprintue.com/blueprint/ovfeibwn/)

<details>
<summary>RotInterpExampleCode</summary>

 ```cpp
FRotator UCommonFunctions::SmoothClampRotation(FRotator InRotator, FRotator ReferenceRotation, float DeltaTime, UPARAM(ref) double& Alpha, float& OutClampStrength, float InnerRadius = 15, float OuterRadius = 25, float ClampStrengthMultiplier = 1)
{

    OutClampStrength = 0;
    //vector of inrot
    FVector v = (InRotator.Quaternion()*ReferenceRotation.Quaternion().Inverse()).Euler();

    //within bounds
    if (v.Size() <= InnerRadius) {
        return InRotator;
    }

    float factor = (OuterRadius - InnerRadius) / FMath::Clamp(OuterRadius - v.Size(), 1, 100);

    FQuat QuatInRotator = InRotator.Quaternion()*ReferenceRotation.Quaternion().Inverse();
    FQuat QuatInverseIn = QuatInRotator.Inverse().GetNormalized();
    FQuat QuatScaledToMinRadius = FQuat(QuatInRotator.GetRotationAxis(), FMath::DegreesToRadians(InnerRadius));
    FQuat QuatScaledToMaxRadius = FQuat(QuatInRotator.GetRotationAxis(), FMath::DegreesToRadians(OuterRadius));

    //Recalculate InverseLerp
    Alpha = UKismetMathLibrary::NormalizeToRange(v.Size(), InnerRadius, OuterRadius);

    //Interp To Target
    Alpha = FMath::Clamp(FMath::FInterpTo(Alpha, 0, DeltaTime, ClampStrengthMultiplier),0,1);

    //for use to ex. scale input speed
    OutClampStrength = Alpha;

    //Slerped from outer towards inner
    FQuat T = FQuat::Slerp(QuatScaledToMinRadius, QuatScaledToMaxRadius, Alpha);

    return (T*ReferenceRotation.Quaternion()).Rotator();
}
```

</details>

**Implementation demo:**
<table>
  <tr>
    <td><img src="Images\SmoothOrbit_demo.gif"/></td>
    <td><img src="Images\OrbitSmoothClampBlueprint_demo.png" /></td>
  </tr>
</table>

The custom nodes used are in a separate repo, linked above (:

## - Adaptable Camera Slerping

Using the earlier mentioned rotation methods, it's possible to dynamically interpolate between freecam angles, and based on preset, assign properties such as limited axis angles, allowed rotation modes, or fading.
This demo video showcases some of theese bahaviors:


https://github.com/GBaath/Portfolio/assets/113012271/c0fec4f4-c4de-482c-9697-246bbcc294bc



---
A dynamic overview of specified areas was planned (but sadly scrapped) using [**this solution**](https://github.com/GBaath/MeshVectorGeneratorUnreal/tree/main) for dynamically generating interpolation points within ediotr specified bounds using meshes.

<img src="Images\MeshVecGenScreenshot.PNG" width="50%"/>


## - Integration with custom preset system

As the built in variant manager is a bit lacking, especially when it comes to event calls when changing variants, we hade to make our own, there are a lot of different subgraphs, but the following should giva a *decent* overview of whats going on.

[**Angle Setting Example**](https://blueprintue.com/blueprint/-g9zlyns/)

[**Controls overview**](https://blueprintue.com/blueprint/873z-46x/)
