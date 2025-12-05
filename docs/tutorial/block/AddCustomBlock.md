# 添加自定义方块

> 方块是世界的基石，从这里开始

## 注册方块与基础属性

在**register**软件包（例如`java/com/sanjin/tutorial/register`）下新建一个文件：**ModBlocks.java**。

先创建一个简单方块：`tutorial_block`，并为它生成对应的方块物品。

这次我们需要多注册一个静态方法，因为每一个方块都需要一个对应的“物品”。我们要确保这两者是同时注册的：

```
public class ModBlocks {

    public static final DeferredRegister.Blocks BLOCKS = DeferredRegister.createBlocks(Tutorial.MODID);

    public static final DeferredBlock<Block> TUTORIAL_BLOCK = BLOCKS.register("tutorial_block",
            registryName -> new Block(BlockBehaviour.Properties.of()
                    .setId(ResourceKey.create(Registries.BLOCK, registryName))
                    .strength(3.0f, 6.0f)      // 硬度等级与抗爆等级
                    .sound(SoundType.STONE)                                // 挖掘时的声音类型
                    .lightLevel(state -> 10)                     // 发光等级（可选）
                    .requiresCorrectToolForDrops()                         // 需要合适工具才能掉落对应的物品（可选）
            )
    );

    public static void registerBlockItems() {
        ModItems.ITEMS.registerSimpleBlockItem("tutorial_block", TUTORIAL_BLOCK, new Item.Properties());
    }

    public static void register(IEventBus bus) {
        BLOCKS.register(bus);
        registerBlockItems();
    }

}
```

将`register()`在主类构造函数中调用，和物品注册的方式一致。常用属性：
- `.strength(硬度, 抗爆)`：控制挖掘时间和爆炸抗性。
- `.sound(SoundType.XXX)`：控制放置/破坏声音。
- `.requiresCorrectToolForDrops()`：要求正确工具才能掉落（如石镐挖石头）。
- `.lightLevel(state -> 10)`：可选，发光等级。
- `.noOcclusion()`：可选，不阻挡视线（玻璃式方块）。

## 模型与状态映射：最简单的立方体

资源路径示例（都在`src/main/resources/assets/tutorial/`下）：
- 贴图：`textures/block/tutorial_block.png`
- 方块模型：`models/block/tutorial_block.json`
- 方块物品模型：`models/item/tutorial_block.json`
- 方块状态：`blockstates/tutorial_block.json`

关于使用DataGen生成普通方块的模型与方块状态文件的方法，请查看：*数据生成(DataGen)*文档

方块模型（六面同贴图）：
```
{
  "parent": "minecraft:block/cube_all",
  "textures": {
    "all": "tutorial:block/tutorial_block"
  }
}
```

方块物品模型直接复用方块模型：
```
{
  "parent": "tutorial:block/tutorial_block"
}
```

方块状态（只有一个变体时很简单）：
```
{
  "variants": {
    "": { "model": "tutorial:block/tutorial_block" }
  }
}
```

## 旋转/朝向/多状态方块

如果方块需要朝向（例如熔炉类方块），可以新增方向属性。示例：在自定义方块类中加入`FACING`并实现旋转/镜像：

```
public class TutorialFurnaceBlock extends HorizontalDirectionalBlock {

    public static final DirectionProperty FACING = BlockStateProperties.HORIZONTAL_FACING;

    public TutorialFurnaceBlock() {
        super(BlockBehaviour.Properties.of()
                .strength(3.5f)
                .sound(SoundType.STONE)
                .requiresCorrectToolForDrops());
        registerDefaultState(this.stateDefinition.any().setValue(FACING, Direction.NORTH));
    }

    @Override
    protected void createBlockStateDefinition(StateDefinition.Builder<Block, BlockState> builder) {
        builder.add(FACING);
    }

    @Override
    public BlockState getStateForPlacement(BlockPlaceContext ctx) {
        return this.defaultBlockState().setValue(FACING, ctx.getHorizontalDirection().getOpposite());
    }

    @Override
    public BlockState rotate(BlockState state, Rotation rotation) {
        return state.setValue(FACING, rotation.rotate(state.getValue(FACING)));
    }

    @Override
    public BlockState mirror(BlockState state, Mirror mirror) {
        return rotate(state, mirror.getRotation(state.getValue(FACING)));
    }
}
```

对应方块状态文件（根据朝向选择模型）：
```
{
  "variants": {
    "facing=north": { "model": "tutorial:block/tutorial_furnace" },
    "facing=south": { "model": "tutorial:block/tutorial_furnace", "y": 180 },
    "facing=west":  { "model": "tutorial:block/tutorial_furnace", "y": 270 },
    "facing=east":  { "model": "tutorial:block/tutorial_furnace", "y": 90 }
  }
}
```

如果还有“点亮/熄灭”等多状态，可以在模型里拆分两份（如`tutorial_furnace_on` / `tutorial_furnace_off`），然后在`blockstates`中根据额外属性选择模型。

### 使用DataGen生成朝向/多状态的 blockstate 与模型

手写blockstate与模型容易出错，推荐用`BlockStateProvider`生成。示例：

```
public class ModBlockStateProvider extends BlockStateProvider {
    public ModBlockStateProvider(PackOutput output, ExistingFileHelper helper) {
        super(output, Tutorial.MODID, helper);
    }

    @Override
    protected void registerStatesAndModels() {
        // 带朝向的熔炉：自动生成 blockstate 变体并复用同一模型
        horizontalBlock(ModBlocks.TUTORIAL_FURNACE.get(),
                state -> models().orientable(
                        name(ModBlocks.TUTORIAL_FURNACE.get()),
                        modLoc("block/tutorial_furnace_side"),
                        modLoc("block/tutorial_furnace_front"),
                        modLoc("block/tutorial_furnace_top")));

        // 如果有“点亮/熄灭”两种模型，可根据属性选择模型：
        // horizontalBlock(block, state -> state.getValue(LIT)
        //     ? models().orientable(name(block) + "_on", ...前面同理...)
        //     : models().orientable(name(block) + "_off", ...前面同理...));

        // 别忘了为方块物品生成模型（复用block模型）
        simpleBlockItem(ModBlocks.TUTORIAL_FURNACE.get(),
                models().orientable(name(ModBlocks.TUTORIAL_FURNACE.get()),
                        modLoc("block/tutorial_furnace_side"),
                        modLoc("block/tutorial_furnace_front"),
                        modLoc("block/tutorial_furnace_top")));
    }
}
```

在数据生成入口`gatherData`中注册：
```
generator.addProvider(true, new ModBlockStateProvider(output, fileHelper));
```

运行`./gradlew runData`后，会在`build/generated`中生成`blockstates/<name>.json`以及对应的`models/block/*.json`和方块物品模型，无需手写，减少命名与朝向出错的风险。

## 语言文件

在`assets/tutorial/lang/en_us.json`中添加：
```
{
  ...

  "block.tutorial.tutorial_block": "Tutorial Block",
  "block.tutorial.tutorial_furnace": "Tutorial Furnace"

  ...
}
```
以及`zh_cn.json`文件中添加：
```
{
  ...

  "block.tutorial.tutorial_block": "教程用方块",
  "block.tutorial.tutorial_furnace": "教程用熔炉"

  ...
}
```

如果有多状态的命名（例如点亮/熄灭），保持同一个翻译键即可；显示的名称不会因为状态改变而变化，除非你在代码中自行更换描述。
