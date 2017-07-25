---
layout:     keynote
title:      "UE4 Foliage Interactive and Fire Propagate"
subtitle:   "UE4 Foliage Interactive and Fire Propagate on terrain and mesh"
iframe:     ""
date:       2017-07-25
author:     "Windy"
tags:
    - game logic
---

Foliage Interactive通过两种方式： 
**1.一个是shader代码里面偏移World Position实现，这种方式简单，效率也不错。
**2.通过在植物里面加入骨骼，进行物理的碰撞检测，这种方式会精确很多，但是需要绑定骨骼 
神海4里面是这两张方式都集合了，大型植物采用绑定骨骼的方式，地面的草丛采用简单的偏移World Position的方式。  

方法1的伪代码：
```
Get player pos
Get dir vector from player pos to grass pos.
Get distance between player pos and grass pos as distance weight.
Get hard weight from uv value.(top will be greater weight.)
The world pos value = dir_vector * (1 - distance_weight) * (uv_hard_weight).
```

UE4的火传播算法，基于Far cry2的算法，http://jflevesque.com/2012/12/06/far-cry-how-the-fire-burns-and-spreads/ 

为了简便，直接将场景分成了40 x 40的格子，并没有做local cell grid的动态合并和分割，大型沙盒场景必须要做分割。 

在UE4里面，用分割的cell跟场景的物体做overlap，获取的Foliage上面附加火焰特效

UE4 overlap代码片段: 
```
FCollisionQueryParams query_params;
TArray<FOverlapResult> ret_array;
FCollisionShape box;
box.SetBox(FVector(fire_data.radius, fire_data.radius, fire_data.radius));
/*const FName TraceTag("MyTraceTag"); // for debug
GetWorld()->DebugDrawTraceTag = TraceTag;
query_params.TraceTag = TraceTag;*/
//FVector box_pos = fire_data.pos - FVector(TerrainWidth / 2, TerrainHeight / 2, 0);
bool on_hit = GetWorld()->OverlapMultiByChannel(ret_array, fire_data.pos, FQuat::Identity, ECollisionChannel::ECC_GameTraceChannel2, box, query_params);
if (ret_array.Num() > 0)
{
    UInstancedStaticMeshComponent *ic = dynamic_cast<UInstancedStaticMeshComponent*>(ret_array[0].GetComponent());
    if (ic)
    {
        FBox fbox = FBox::BuildAABB(fire_data.pos, FVector(fire_data.radius, fire_data.radius, fire_data.radius));
        TArray<int32> ins_indice = ic->GetInstancesOverlappingBox(fbox);
        for (int i = 0; i < ins_indice.Num(); ++i)
        {
            UE_LOG(LogAssetManagerEditor, Log, TEXT("Hit instance index is %d\n"), ins_indice[i]);

            FTransform inst_trans;
            ic->GetInstanceTransform(ins_indice[i], inst_trans, true);

            //UParticleSystemComponent* ParticleTemp;
            UParticleSystemComponent* ParticleTemp = UGameplayStatics::SpawnEmitterAttached(FireParticle,
                RootComponent,
                NAME_None,
                inst_trans.GetTranslation(),
                inst_trans.GetRotation().Rotator(),
                EAttachLocation::KeepWorldPosition,
                false);

            if (ParticleTemp)
            {
                ParticleTemp->bVisible = true;
                //ParticleTemp->SetRelativeScale3D(FVector(10.0f, 10.0f, 10.0f));
                ParticleTemp->SetWorldScale3D(FVector(10.0f, 10.0f, 10.0f));
                ParticleTemp->Activate(true);
                ParticleTemp->ActivateSystem();
            }
        }
    }
}
```

基于Mesh的火传播 

**1. 根据物体的bounding box，平均分割变成三维的cell，然后通过cell与mesh的overlap获取有效的cell。
**2. 往最近的cell进行传播

(从这个教程里面看到，如果要精确的话，还可以在cell里面跟mesh做ray cast获取mesh表面精确位置来spawn火焰粒子，火焰传播作者也使用了Djikstra's shortest path algorithm来进行火焰生成路径的选择 https://www.sanathshekar.com/grid-generator)

### Power by [windy](http://windypaper.github.io)
