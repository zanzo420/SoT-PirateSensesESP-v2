# SoT-PirateSensesESP-v2
Sea of Thieves "Pirate Senses" ESP (External Overlay) 


[B]Sig :[/B]
[CODE]//LineOfSight Unicode
#define OFFS_VISIBLE 0x128BEE0
//48 8B 1D ?? ?? ?? ?? 48 85 DB 74 3B 41
#define OFFS_UWORLD 0x409AE80
//48 89 5C 24 ? 48 89 74 24 ? 57 48 81 EC ? ? ? ? 41 0F B6 F9
#define OFFS_W2S 0x15BAAB0
//
#define OFFS_GNAMES 0x3F8B490
//E8 ? ? ? ? F3 0F 10 48 ? F3 0F 10 40
#define OFFS_GETBONEMATRIX 0x14F3660[/CODE]

[B]World to Screen :[/B]
[CODE]bool WScreen(APlayerController* m_Player, FVector WorldPosition, FVector2 *ScreenPosition)
{
	static DWORD_PTR funcAddress = 0;
	if (!funcAddress)funcAddress = (DWORD_PTR)(DWORD64)Module(L"PUBGLite-Win64-Shipping.exe") + OFFS_W2S;
	if (funcAddress != 0)
	{
		return reinterpret_cast<char(__fastcall*)(APlayerController*, FVector, FVector2 *, char)>(funcAddress)(m_Player, WorldPosition, ScreenPosition, 0);
	}
}[/CODE]
[B]Usage :[/B]
[CODE]FVector root;
FVector2 rootOut;
GetBoneLocation(Actor->p_mesh(), &root, 0);

bool rootScreen = WScreen(LocalPlayer->p_playerController(), root, &rootOut);
if (rootScreen)
{
  DrawText("blablabla", rootOut.X, rootOut.Y);
}[/CODE]

[B]VisibleCheck :[/B]
[CODE]bool LineOfSightTo(APlayerController * thiz, APawn *Other, FVector *ViewPoint)
{
	static DWORD_PTR funcAddress = 0;
	if (!funcAddress)funcAddress = (DWORD_PTR)(DWORD64)Module(L"PUBGLite-Win64-Shipping.exe") + OFFS_VISIBLE;
	if (funcAddress) { return reinterpret_cast<int(__fastcall*)(APlayerController * thiz, APawn *Other, FVector *ViewPoint)>(funcAddress)(thiz, Other, ViewPoint); }
}[/CODE]
[B]Usage :[/B]
[CODE]bool IsVisible = LineOfSightTo(LocalPlayer->p_playerController(), m_PL, &Zero);
if (IsVisible == 1) { color = color_esp_human; }
if (IsVisible == 0) { color = color_esp_NoVishuman; }[/CODE]

[B]Bone coord by ID :[/B]
[CODE]
typedef FMatrix *(__thiscall* _GetBoneMatrix)(USkeletalMeshComponent *pThis, FMatrix *result, int BoneIdx);
_GetBoneMatrix pGetBoneMatrix;
FMatrix * GetBoneMatrix(USkeletalMeshComponent *pObj, FMatrix *result, int BoneIdx)
{
	static DWORD_PTR funcAddress = 0;
	if (!funcAddress)funcAddress = (DWORD_PTR)(DWORD64)Module(L"PUBGLite-Win64-Shipping.exe") + OFFS_GETBONEMATRIX;
	if (funcAddress != 0)
	{
		_GetBoneMatrix pGetBoneMatrix = (_GetBoneMatrix)(funcAddress);
		return pGetBoneMatrix(pObj, result, BoneIdx);
	}
}

void GetBoneLocation(USkeletalMeshComponent* pMesh, FVector* vBone, int boneidx)
{
	if (IsBadReadPtr(pMesh, 8))return;

	FMatrix vMatrix;
	FMatrix *vTempvMatrix = GetBoneMatrix(pMesh, &vMatrix, boneidx);
	*vBone = vMatrix.WPlane;//Trans
}
[/CODE]
[B]Usage :[/B]
[CODE]GetBoneLocation(ObjMesh, FVector, BoneId);[/CODE]

[B]Struct :[/B]
[CODE]class cUWorld
{
public:
	ULevel*				p_persistentLevel() { return *(ULevel**)(this + 0x30); }
	UGameInstance*		p_owningGameInstance() { return *(UGameInstance**)(this + 0x150); }

};

class ULevel
{
public:
	char pad_0x0000[0xA0];
	TArray<APawn*>actors;
};

class UGameInstance
{
public:
	char pad_0x0000[0x38]; 
	TArray<ULocalPlayer*>localPlayers;
};

class ULocalPlayer
{
public:
	APlayerController*		p_playerController() { return *(APlayerController**)(this + 0x30); }
	ViewportClient*			p_ViewportClient() { return *(ViewportClient**)(this + 0x58); }
	FVector*				v_Position() { return (FVector*)(this + 0x70); }
};

class APlayerController
{
public:
	float					f_Pitch() { return *(float*)(this + 0x3B4); }
	float					f_Yaw() { return *(float*)(this + 0x3B8); }
	APawn*					p_LocalPawn() { return *(APawn**)(this + 0x3F0); }
	Camera*					p_Camera() { return *(Camera**)(this + 0x410); }
};

class ViewportClient
{
public:
	World*					p_World() { return *(World**)(this + 0x80); }
};

class APawn
{
public:
	__int32					ID() { return *(__int32*)(this + 0x18); }
	APawn*					p_pawn() { return *(APawn**)(this + 0x148); }
	USceneComponent*		p_rootComp() { return *(USceneComponent**)(this + 0x160); }
	APlayerState*			p_PlayerState() { return *(APlayerState**)(this + 0x388); }
	USkeletalMeshComponent*	p_Mesh() { return *(USkeletalMeshComponent**)(this + 0x3C8); }
	FString*                s_PlayerName() { return (FString*)(this + 0x788); }
	__int32					i_TeamID() { return *(__int32*)(this + 0x79C); }
	__int32					i_TeamNum() { return *(__int32*)(this + 0xAD8); }
	float					f_Health() { return *(float*)(this + 0x1474); }
	float					f_HealthMax() { return *(float*)(this + 0x147C); }
};[/CODE]
