# 添加自定义的工具与武器

> 武器决定战斗手感，工具决定生存效率

本节将带你从最基础的“套用原版工具材料注册一把剑”，到“自定义工具材料再注册整套工具”。阅读完之后，你应该能为模组添加自己的武器、镐子、斧头、锹与锄头，并为它们配置耐久、挖掘等级、攻击力和修复材料。

## 准备文件位置

和之前注册物品的流程一样，我们依然在**register**软件包（例如`java/com/sanjin/tutorial/register`）中的**ModItems.java**里注册工具与武器。如果你习惯将物品类分散到子包，也可以新建一个`weapon`或`tool`子包存放特殊的物品类。

## 直接使用原版 ToolMaterial 注册

从 Minecraft 1.21.3 开始，工具/武器的材料数据存放在`ToolMaterial`中。如果你想快速得到一把“与钻石级别相同”的武器或工具，可以直接使用原版的`ToolMaterial.DIAMOND`等常量。下面以自定义一把紫水晶剑为例：

```
public class ModItems {

    ...

    public static final Supplier<Item> AMETHYST_SWORD = ITEMS.register("amethyst_sword",
            () -> new SwordItem(
                    ToolMaterial.DIAMOND, // 基础耐久/挖掘等级/附魔性参考钻石
                    3,                    // 额外攻击伤害
                    -2.2f,                // 攻击速度（负数越大，挥舞越慢）
                    new Item.Properties()
                            .durability(1234) // 自定义耐久，覆盖Tier默认耐久
                            .fireResistant()  // 可选：掉进岩浆不被烧毁
            )
    );

    ...

}
```

关键参数说明：
- `ToolMaterial.DIAMOND`：决定基础耐久、附魔性、挖掘等级。你可以换成`IRON`、`NETHERITE`等。
- 第二个参数（额外攻击伤害）与第三个参数（攻击速度）会在基础值上叠加，推荐参考原版武器的手感进行调整。
- `durability()`可以覆盖Tier自带耐久，`fireResistant()`决定物品是否被熔毁。

同理，你可以用`PickaxeItem`、`AxeItem`、`ShovelItem`、`HoeItem`来快速注册一整套工具，只需调整攻击伤害/攻速的两个浮点参数即可。

## 自定义自己的 ToolMaterial

当原版工具材料无法满足你的需求（例如要做一把“挖掘等级和下界合金一样，但更耐用、更易附魔”的工具），就需要创建自定义的`ToolMaterial`。

### 编写自定义工具材料

`ToolMaterial`是一个`record`，构造参数顺序为：

```
ToolMaterial(
    TagKey<Block> incorrectBlocksForDrops, // 无法正确采集的方块标签
    int durability,                        // 耐久
    float speed,                           // 挖掘速度
    float attackDamageBonus,               // 额外攻击伤害
    int enchantmentValue,                  // 附魔性
    TagKey<Item> repairItems               // 修复材料的物品标签
)
```

你可以直接在一个工具类中定义自定义材料，打开**register**软件包，让我们新建一个Java类，命名为`ModToolMaterials`：

```
public class ModToolMaterials {

    public static final ToolMaterial TUTORIAL_MATERIAL = new ToolMaterial(
            BlockTags.INCORRECT_FOR_NETHERITE_TOOL, // 以更高等级为基准，能挖掘所有下界合金能挖的方块
            2048,                                    // 耐久
            9.5f,                                    // 挖掘速度
            4.0f,                                    // 额外攻击伤害
            20,                                      // 附魔性
            ItemTags.DIAMOND_TOOL_MATERIALS          // 修复材料标签，示例用钻石修
    );
}
```

如果你想用自定义修复材料，可以在数据包中创建自己的物品标签，然后把`ItemTags.DIAMOND_TOOL_MATERIALS`替换为你的自定义标签。

#### 在数据包中创建自定义物品标签

1. 打开资源目录：`src/main/resources/data`。如果没有`data`文件夹，请自行创建。
2. 在`data/<你的modid>/tags/items/`下新建一个`.json`文件，命名为你想要的标签名，例如：`tutorial_tool_repair.json`。
3. 写入内容：
```
{
  "replace": false,
  "values": [
    "tutorial:amethyst_ingot",   // 你的修复材料
    "#minecraft:diamond_tool_materials" // 也可以包含已有标签（可选）
  ]
}
```
`replace:false`表示不会覆盖同名标签的其他来源，`values`可以混合物品ID和其他标签（用#前缀）。
4. 在代码里导入你的标签：
```
public static final TagKey<Item> TUTORIAL_TOOL_REPAIR =
        TagKey.create(Registries.ITEM, new ResourceLocation("tutorial", "tutorial_tool_repair"));
```
然后把`ToolMaterial`构造器的最后一个参数替换为`TUTORIAL_TOOL_REPAIR`即可。

### 用自定义 ToolMaterial 注册工具/武器

现在在**ModItems**里使用`TUTORIAL_MATERIAL`注册一整套工具：

```
public class ModItems {

    ...

    public static final Supplier<Item> TUTORIAL_SWORD = ITEMS.register("tutorial_sword",
            () -> new SwordItem(ModToolMaterials.TUTORIAL_MATERIAL, 4, -2.0f,
                    new Item.Properties().durability(2048)));

    public static final Supplier<Item> TUTORIAL_PICKAXE = ITEMS.register("tutorial_pickaxe",
            () -> new PickaxeItem(ModToolMaterials.TUTORIAL_MATERIAL, 1, -2.6f,
                    new Item.Properties().durability(2048)));

    public static final Supplier<Item> TUTORIAL_AXE = ITEMS.register("tutorial_axe",
            () -> new AxeItem(ModToolMaterials.TUTORIAL_MATERIAL, 5.0f, -3.0f,
                    new Item.Properties().durability(2048)));

    public static final Supplier<Item> TUTORIAL_SHOVEL = ITEMS.register("tutorial_shovel",
            () -> new ShovelItem(ModToolMaterials.TUTORIAL_MATERIAL, 1.5f, -3.0f,
                    new Item.Properties().durability(2048)));

    public static final Supplier<Item> TUTORIAL_HOE = ITEMS.register("tutorial_hoe",
            () -> new HoeItem(ModToolMaterials.TUTORIAL_MATERIAL, -4, 0.0f,
                    new Item.Properties().durability(2048)));

    ...

}
```

几点小贴士：
- 同一把工具的`durability()`通常与工具材料耐久保持一致，但也可以单独调整以营造差异感。
- 斧头的“额外攻击伤害”和“攻速”通常比剑更高、更慢；锄头/锹的数值更低，可参考原版。
- 如果你还没引入自定义创造物品栏，请记得在已有的创造物品栏配置里把新物品加进去，方便在游戏内获取。

## 资源文件与语言文件

和普通物品一样，工具/武器需要：
- 语言键：`item.tutorial.tutorial_sword` 等，写入`assets/tutorial/lang/en_us.json`与`zh_cn.json`。
- 模型文件：`assets/tutorial/models/item/tutorial_sword.json`等，通常父模型使用`"parent": "item/handheld"`（武器/工具类）或`"item/handheld_rod"`，贴图路径放在`assets/tutorial/textures/item/`下。

示例模型：
```
{
  "parent": "item/handheld",
  "textures": {
    "layer0": "tutorial:item/tutorial_sword"
  }
}
```

## 调试与常见问题

- **无法挖掘某些方块**：检查自定义Tier的方块标签是否包含对应的矿物标签（如`NEEDS_DIAMOND_TOOL`或自定义标签），以及工具类型是否正确（镐挖石头、铲挖泥土）。
- **攻击力/攻速手感不对**：多参考原版数值，或在游戏里用`/attribute`查看现有武器的数值作为对比。
- **语言键显示注册名**：确认物品注册名与语言文件键一致，JSON 逗号与括号格式正确。
- **贴图紫黑块**：确认贴图文件名与模型引用路径一致，路径全部小写且没有空格。

