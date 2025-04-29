# Play Montage Advanced

Provides the ability task node `Play Montage Advanced and Wait` to play multiple montages on multiple meshes.

Supports:
 * Montage Replication
 * Event Tags
 * Psuedo Notifies that are much more reliable than anim based notifies
 * Override blend-in settings programmatically
 * Driver and Driven montages (Driven montages can match the animation duration of the Driver montage)

> [!TIP]
> If you want to use the interface version with blueprint, you need to add this manually by adding a `BlueprintImplementableEvent` that `IPlayMontageByTagInterface::GetAbilityMontagesByTag` calls, otherwise C++ is required.

![UnrealEditor-Win64-DebugGame_2025-04-30_05-11-05](https://github.com/user-attachments/assets/20ab9007-2963-4729-92b6-8b6ced9d288c)

## Pseudo notify states

Add a `AnimNotifyStateByTag` or `AnimNotifyByTag` to your montage and assign a gameplay tag

![image](https://github.com/user-attachments/assets/577a6180-a648-42d5-a659-94a9f389a0c7)

![image](https://github.com/user-attachments/assets/a7746292-171a-4958-9f87-92b5d9a0cefc)

Then use the callback on the node to respond to the notify or notify state

![image](https://github.com/user-attachments/assets/f13fa2fb-fc70-4797-9f29-ac0c1031d4dc)


## Setup

### Project

You will need to include `"PlayMontageAdvanced"` in both your .uproject and your `Build.cs` `PublicDependencyModuleNames`


### Ability System

Either use `UPlayMontageAbilitySystemComponent` or derive your own `UAbilitySystemComponent` from it.

Derive your gameplay abilities from `UPlayMontageGameplayAbility`.

### Optional Actor Interface

> [!IMPORTANT]
> Only required if using the interface option, with `MontageTag` on the node

Implement the `IPlayMontageByTagInterface` interface to your Avatar Actor

Override `GetAbilityMontagesByTag`:

```cpp
bool AMyCharacterPlayer::GetAbilityMontagesByTag(const FGameplayTag& MontageTag, UAnimMontage*& DriverMontage,
	FDrivenMontages& DrivenMontages) const
{
	DriverMontage = ThirdPersonBodyMontage;
	DrivenMontages.Reset();

	// These montages will play for all
	DrivenMontages.DrivenMontages.Add({ ThirdPersonWeaponMontage, WeaponMeshTP });

	// These montages only play locally, as per UPlayMontageByTagLib::ShouldPlayLocalDrivenMontages()
	DrivenMontages.LocalDrivenMontages.Add({ FirstPersonBodyMontage, BodyMeshFP }, { FirstPersonWeaponMontage, WeaponMeshFP });
	
	// Returning false means no montage will play at all, it will abort
	return true;
}
```

Any montages added to `LocalDrivenMontages` will only play if this is true:

```cpp
bool UPlayMontageByTagLib::ShouldPlayLocalDrivenMontages(const AActor* AvatarActor)
{
	const APawn* Pawn = AvatarActor ? Cast<APawn>(AvatarActor) : nullptr;
	APlayerController* PlayerController = Pawn ? Pawn->GetLocalViewingPlayerController() : nullptr;
	return PlayerController != nullptr;
}
```

Now you can add a `PlayMontageAdvancedAndWait` node to your ability blueprint and it should work.

Your next step will be to pass in a `FGameplayTag` for `MontageTag` and factor that in for `GetAbilityMontagesByTag`. For example `MontageTag.Weapon.SMG.Reload`.

## Notes
Code was used from [GASShooter](https://github.com/tranek/GASShooter/)

## Changelog

### 1.4.0
* Add support for params, interface, or component

### 1.3.0
* Refactor to PlayMontageAdvanced
* Remove dependency on interface, allowing params to be used instead

### 1.2.0
* Fix bug where montage wasn't being stopped when ability ends
* Added support for notify by tag that is processed in the GA itself

### 1.1.0
* Added driven montage replication
* Added blend in overrides
* Added event tag callback
* Added option to scale driven montages to driver duration

### 1.0.0
* Release
