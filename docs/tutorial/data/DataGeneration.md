# 数据生成（DataGen）——让JSON自动化

> 不再手搓大堆JSON，交给datagen自动完成

手写json文件实在是太麻烦了！而且容易出错。没关系，学完这篇文档，DataGen会为你解决一切问题！

我们可以用官方的数据生成器一次性产出配方、掉落表、标签等资源文件（本章不生成语言文件，容易出现意料之外的错误）。本节将按顺序讲解：**生成物品模型 → 生成方块模型 → 生成方块状态 → 生成配方 → 修改原版战利品列表**，并在IDE中直接运行`runData`。后面附加了自定义掉落表、标签的生成示例，按需阅读。

## 准备：添加DataGen入口

在你的主mod包里创建一个数据生成入口类（如：`java/com/sanjin/tutorial/datagen/ModDataGenerators.java`），然后写入以下内容：

```
@EventBusSubscriber(modid = Tutorial.MODID)
public class ModDataGenerators {

    @SubscribeEvent
    public static void gatherData(GatherDataEvent event) {
        DataGenerator generator = event.getGenerator();
        PackOutput output = generator.getPackOutput();
        CompletableFuture<HolderLookup.Provider> lookupProvider = event.getLookupProvider();
        ExistingFileHelper fileHelper = event.getExistingFileHelper();

        // 以下代码请在你完成相应的java类后再写入！例如写完ModRecipeProvider.java文件后，再将它添加到if语句中
        if (event.includeServer()) {
            generator.addProvider(true, new ModRecipeProvider(output, lookupProvider));
            generator.addProvider(true, new ModLootTableProvider(output, lookupProvider));
            generator.addProvider(true, new ModBlockTagsProvider(output, lookupProvider, fileHelper));
            generator.addProvider(true, new ModItemTagsProvider(output, lookupProvider, fileHelper));
            generator.addProvider(true, new ModGlobalLootModifierProvider(output)); // 修改原版战利品
        }
        
    }
}
```

在IDEA中运行`runData`（Gradle任务），或命令行`./gradlew runData`，生成的内容会输出到`build/generated`，随后由资源刷新到`src/generated`。已有的文件经过一次`runData`命令后会保留，后续添加新内容时需要重新运行`runData`。

## 生成物品模型文件

物品模型通常是平面(`item/generated`)或手持(`item/handheld`)。使用`ItemModelProvider`自动生成：

```
public class ModItemModelProvider extends ItemModelProvider {
    public ModItemModelProvider(PackOutput output, ExistingFileHelper helper) {
        super(output, Tutorial.MODID, helper);
    }

    @Override
    protected void registerModels() {
        // 平面物品（父模型 item/generated）
        basicItem(ModItems.TUTORIAL_CORE.get());

        // 手持类（父模型 item/handheld）
        handheld(ModItems.TUTORIAL_SWORD.get());
    }
}
```

然后注册到DataGen。注意！贴图需放在`assets/<modid>/textures/item/`，如果写入DataGen的物品缺少贴图，运行DataGen时会报错！

## 生成方块模型文件

方块模型与方块状态通常由`BlockStateProvider`统一生成，这里先关注方块模型（`models/block/*.json`）：

```
public class ModBlockStateProvider extends BlockStateProvider {
    public ModBlockStateProvider(PackOutput output, ExistingFileHelper helper) {
        super(output, Tutorial.MODID, helper);
    }

    @Override
    protected void registerStatesAndModels() {
        // 简单立方体（六面同贴图）
        simpleBlock(ModBlocks.TUTORIAL_BLOCK.get());

        // 矿石同理
        simpleBlock(ModBlocks.TUTORIAL_ORE.get());
    }
}
```

常用方法：
- `simpleBlock(Block)`：生成`blockstates/<name>.json`与`models/block/<name>.json`（cube_all）。
- `cubeAll(Block)`：可与`simpleBlockItem`结合，为方块物品生成模型。

## 生成方块状态文件

`BlockStateProvider`也会生成`blockstates/<name>.json`。如需朝向/多状态，可用`horizontalBlock`、`axisBlock`等方法：

```
@Override
protected void registerStatesAndModels() {
    horizontalBlock(ModBlocks.TUTORIAL_FURNACE.get(),
            state -> models().orientable(
                    name(ModBlocks.TUTORIAL_FURNACE.get()),
                    modLoc("block/tutorial_furnace_side"),
                    modLoc("block/tutorial_furnace_front"),
                    modLoc("block/tutorial_furnace_top")));

    // 方块有物品形态时，生成物品模型
    simpleBlockItem(ModBlocks.TUTORIAL_BLOCK.get(), cubeAll(ModBlocks.TUTORIAL_BLOCK.get()));
}
```

输出：`blockstates/<name>.json` + `models/block/*.json` + `models/item/*.json`（方块物品）。

如果你的方块具备动画、复杂贴图等等导致使用DataGen生成太麻烦，你可以依然使用Blockbench或其他软件导出的json文件作为该方块的模型/状态文件，放入**resources/assets/tutorial**目录下的**blcokstates**文件夹和**models/block**目录下

放心，mod加载时，DataGen生成的json文件与你手动导入/手写的json文件都会被检测。只要确保同一个物品/方块有唯一的资源文件即可
，否则可能会导致资源加载出错。
## 编写配方生成器（RecipeProvider）

新建`ModRecipeProvider`继承`RecipeProvider`：

```
public class ModRecipeProvider extends RecipeProvider {

    public ModRecipeProvider(PackOutput output, CompletableFuture<HolderLookup.Provider> lookupProvider) {
        super(output, lookupProvider);
    }

    @Override
    protected void buildRecipes(RecipeOutput output) {
        ShapedRecipeBuilder.shaped(RecipeCategory.COMBAT, ModItems.TUTORIAL_SWORD.get())
                .define('X', Items.AMETHYST_SHARD)
                .define('S', Items.STICK)
                .pattern("X")
                .pattern("X")
                .pattern("S")
                .unlockedBy("has_amethyst", has(Items.AMETHYST_SHARD))
                .save(output);

        ShapelessRecipeBuilder.shapeless(RecipeCategory.MISC, ModItems.TUTORIAL_CORE.get())
                .requires(ModItems.TUTORIAL_SHARD.get(), 4)
                .requires(Items.NETHER_STAR)
                .unlockedBy("has_shard", has(ModItems.TUTORIAL_SHARD.get()))
                .save(output, new ResourceLocation(Tutorial.MODID, "tutorial_core"));
    }
}
```

说明：
- `RecipeOutput` 会写入 `data/<modid>/recipes/*.json`。
- `RecipeCategory` 决定创造标签分类（影响配方书分组）。
- 如果要“修改”原版配方，可以用相同的`ResourceLocation("minecraft","<recipe_id>")`覆盖它（谨慎，以免与其他mod冲突）。

## 修改原版战利品列表（全局战利品修改器）

当你想在原版战利品基础上“追加/替换”内容，而不是重写整个表，可以使用`GlobalLootModifierProvider`：

```
public class ModGlobalLootModifierProvider extends GlobalLootModifierProvider {
    public ModGlobalLootModifierProvider(PackOutput output) {
        super(output, Tutorial.MODID);
    }

    @Override
    protected void start() {
        add("add_core_to_zombie",
                new AddItemModifier(
                        new LootItemCondition[]{
                                LootTableIdCondition.builder(new ResourceLocation("minecraft", "entities/zombie")).build()
                        },
                        ModItems.TUTORIAL_CORE.get()
                ));
    }
}
```

说明：
- Provider在`data/<modid>/loot_modifiers/`生成JSON，并在`global_loot_modifiers.json`里注册。
- `AddItemModifier`等类位于`net.neoforged.neoforge.common.loot`包（2.0.115可用）。你也可以自定义`LootModifier`实现更复杂逻辑。
- 修改原版战利品是叠加行为，不会完全覆盖表；如需彻底替换，请直接生成同名loot table覆盖。

## 附加：编写自定义掉落表（LootTableProvider）

1. 创建主Provider：
```
public class ModLootTableProvider extends LootTableProvider {
    public ModLootTableProvider(PackOutput output, CompletableFuture<HolderLookup.Provider> lookupProvider) {
        super(output, Set.of(), List.of(
                new SubProviderEntry(ModBlockLoot::new, LootContextParamSets.BLOCK),
                new SubProviderEntry(ModEntityLoot::new, LootContextParamSets.ENTITY)
        ), lookupProvider);
    }
}
```
2. 方块掉落示例（继承`BlockLootSubProvider`）：
```
public class ModBlockLoot extends BlockLootSubProvider {
    public ModBlockLoot() {
        super(Set.of(), FeatureFlags.REGISTRY.allFlags());
    }

    @Override
    protected void generate() {
        dropSelf(ModBlocks.TUTORIAL_BLOCK.get());
        add(ModBlocks.TUTORIAL_ORE.get(), block -> createOreDrop(block, ModItems.RAW_TUTORIAL.get()));
    }

    @Override
    protected Iterable<Block> getKnownBlocks() {
        return ModBlocks.BLOCKS.getEntries().stream().map(RegistryObject::get).toList();
    }
}
```
3. 实体掉落示例（继承`EntityLootSubProvider`）：
```
public class ModEntityLoot extends EntityLootSubProvider {
    public ModEntityLoot() {
        super(FeatureFlags.REGISTRY.allFlags());
    }

    @Override
    public void generate() {
        add(ModEntities.TUTORIAL_MOB.get(), LootTable.lootTable()
                .withPool(LootPool.lootPool()
                        .setRolls(ConstantValue.exactly(1))
                        .add(LootItem.lootTableItem(ModItems.TUTORIAL_DROP.get())
                                .apply(SetItemCountFunction.setCount(UniformGenerator.between(1.0f, 3.0f)))
                        )));
    }

    @Override
    protected Stream<EntityType<?>> getKnownEntityTypes() {
        return ModEntities.ENTITIES.getEntries().stream().map(RegistryObject::get);
    }
}
```

生成的文件位于`data/<modid>/loot_tables/...`，进入游戏后自动加载。

## 附加：标签生成（Block/Item Tags）

示例Block标签：
```
public class ModBlockTagsProvider extends BlockTagsProvider {
    public ModBlockTagsProvider(PackOutput output, CompletableFuture<HolderLookup.Provider> lookupProvider, ExistingFileHelper helper) {
        super(output, lookupProvider, Tutorial.MODID, helper);
    }

    @Override
    protected void addTags(HolderLookup.Provider provider) {
        tag(BlockTags.MINEABLE_WITH_PICKAXE).add(ModBlocks.TUTORIAL_BLOCK.get());
        tag(BlockTags.NEEDS_DIAMOND_TOOL).add(ModBlocks.TUTORIAL_ORE.get());
    }
}
```
Item标签继承`ItemTagsProvider`，并可复用方块标签：
```
public class ModItemTagsProvider extends ItemTagsProvider {
    public ModItemTagsProvider(PackOutput output, CompletableFuture<HolderLookup.Provider> lookupProvider, ExistingFileHelper helper) {
        super(output, lookupProvider, new ModBlockTagsProvider(output, lookupProvider, helper), Tutorial.MODID, helper);
    }

    @Override
    protected void addTags(HolderLookup.Provider provider) {
        tag(ItemTags.TOOLS).add(ModItems.TUTORIAL_SWORD.get());
        copy(BlockTags.NEEDS_DIAMOND_TOOL, ItemTags.NEEDS_DIAMOND_TOOL);
    }
}
```

## 运行与输出路径

- 运行：`./gradlew runData` 或在IDEA的Gradle工具窗口执行`runData`。
- 输出：`build/generated` → IDE一般会将其挂载为`src/generated/resources`。提交代码时只需提交生成的资源文件（或保持生成目录受控由CI生成）。
- 常见错误：路径大小写、注册名不一致、`RegistryObject#get`在Provider中使用前必须已注册。
