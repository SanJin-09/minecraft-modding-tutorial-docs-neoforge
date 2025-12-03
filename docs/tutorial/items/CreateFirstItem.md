# 为Minecraft添加你的第一个自定义物品！
> Minecraft三要素：物品（Item），方块（Block），实体（Entity）
## 文件结构
一个优秀的项目必须具备优秀的项目结构，编写Minecraft mod也是一样。从第一个自定义物品开始，我们就要有这样的意识。

打开IDEA并打开你的模版项目，以我的包名com.snajin.tutorial为例，请按照以下顺序找到目标文件夹位置：**src/main/java/com.snajin.tutorial**

你会看到该文件夹目录下已经有了三个Java文件：Config.java，Tutorial.java，TutorialClient.java（后两个Java文件的命名应该与你的mod正式名相同，这里以我的mod：Tutorial为例）。我们需要在这里创建一个新的文件夹。

右键**com.snajin.tutorial**文件夹，选择新建软件包，命名为：**register**。然后在**register**文件夹下新建一个Java类，命名为：**ModItems**。未来所有的自定义物品的注册都需要在这个Java文件中进行。

## 注册第一个物品
下面打开**Moditems.java**文件，请跟着指示敲入代码，我会在之后解释每个语句的作用。
注意！接下来所有类似**Tutorial**，*tutoial*这样的字段都是代指你的mod命名，不要跟着敲入Tutoial，否则会报错。

首先写入下面这段语句：

```
public class ModItems {

    public static final DeferredRegister.Items ITEMS = DeferredRegister.createItems(Tutorial.MODID);

}
```

像我上面说的，`Tutorial.MODID`是你的mod命名，如果你的mod命名为A，那么上面的代码就呀替换成A.MODID。后续所有的Tutorial相关字段都默认替换成你自己的mod命名。

接着在上面的语句后，敲入下面这段代码，来注册我们的第一个自定义物品：custom_item

```
public class ModItems {
    
    public static final DeferredRegister.Items ITEMS = DeferredRegister.createItems(Tutorial.MODID);

    public static final Supplier<Item> CUSTOM_ITEM = ITEMS.registerSimpleItem("custom_item", new Item.Properties());
    
}
```

`CUSTOM_ITEM`与`custom_item`是该自定义物品的命名，你可以替换为你需要的命名。注意大小写。如果你发现你的代码中有任何字段显示为红色报错，不用紧张。将你的光标移动到有红色报错的字段上，在弹出的提示选项中选择**更多操作** -> **导入类**。

接着写入下面这段代码：

```
public class ModItems {

    public static final DeferredRegister.Items ITEMS = DeferredRegister.createItems(Tutorial.MODID);

    public static final Supplier<Item> CUSTOM_ITEM = ITEMS.registerSimpleItem("custom_item", new Item.Properties());

    public static void register(IEventBus eventBus) {
        ITEMS.register(eventBus);
    }

}
```

接下来让我们打开Tutoiral.java文件，你会看到文件里存在繁多的代码字段，不要紧张，让我们把`public class Tutorial{}`内部多余的测试内容和注释**删除**，留下以下内容：
```
public class Tutorial {

    public static final String MODID = "tutorial";

    public static final Logger LOGGER = LogUtils.getLogger();

    public Tutorial(IEventBus modEventBus, ModContainer modContainer) {

        modEventBus.addListener(this::commonSetup);

        NeoForge.EVENT_BUS.register(this);

        modEventBus.addListener(this::addCreative);
        
        modContainer.registerConfig(ModConfig.Type.COMMON, Config.SPEC);
    }

    private void commonSetup(FMLCommonSetupEvent event) {

    }

    private void addCreative(BuildCreativeModeTabContentsEvent event) {

    }

    @SubscribeEvent
    public void onServerStarting(ServerStartingEvent event) {

    }
}
```

现在整个文件看起来整洁多了，让我们找到`public Tutorial(IEventBus modEventBus, ModContainer modContainer) {}`。

然后写入下面这段代码：

```
public Tutorial(IEventBus modEventBus, ModContainer modContainer) {

    //这里只是省略了这个方法中其他的代码，不要将它们删除
    ModItems.ITEMS.register(modEventBus);

}
```

恭喜你，现在已经成功注册了你的第一个物品！下面我将解释我们刚刚写入的代码的实际作用。

`public static final DeferredRegister.Items ITEMS = DeferredRegister.createItems(Tutorial.MODID);
`

这段语句是为了创建一个延迟注册器：*ITEMS*，它会在Minecraft启动的适当时机将物品注册到游戏中，而不是立即注册​（立即注册很容易出现依赖报错）。未来，我们在注册自定义的方块，实体，类型，数据组件，配方，修改战利品列表等等几乎所有的自定义内容都需要先创建这样的一个延迟注册器。

```
public static final Supplier<Item> CUSTOM_ITEM = ITEMS.registerSimpleItem("custom_item", new Item.Properties());

```
这段语句才是真真注册了我们的自定义物品：`custom_item`。这里利用上面提到的延迟注册器ITEMS，并调用了它的方法：registerSimpleItem()注册了一个物品。方法中传入的两个参数：
- "custom_item"：这是物品的注册名称（registry name），这将成为物品在游戏中的唯一标识符
- new Item.Properties()：这是物品的属性配置，你可以进一步设置该物品的最大堆叠属性，是否可食用等等。

```
public static void register(IEventBus eventBus) {
    ITEMS.register(eventBus);
}

```
这段代码是实际触发注册过程的关键步骤，必须在模组的主类构造函数中，也就是Tutorial.java中调用，这也是为什么我们要在Tutorial.java文件中写入`ModItems.ITEMS.register(modEventBus);`段原因。到此为止，`custom_item`这个物品已经被成功注册到了游戏中。

## 完善资源文件
仅仅只是注册这个物品还不够，如果你这个时候进入游戏获取你的自定义物品，你能得到的只是一个带有注册名的紫黑色方块，因为我们的物品还缺少模型文件，语言文件和贴图文件。

### 添加语言文件

现在请找到**src/main/resources**。在这个文件目录下你会发现已经存在了一个**assets/tutorial/lang**的文件目录，在这个目录下存在一个**en_us.json**文件，这就是mod的英语语言文件。

打开**en_us.json**文件，在文件的中括号中写入下面这段内容：

```
{
    "item.tutorial.custom_item":"Custom item"
}
```

`tutorial`是你的mod的注册名，custom_item是你的物品注册名，冒号后面的双引号中填入你的自定义物品的英文名。

如果你发现你的json文件出现了报错，请检查：你的json文件中，是否除了最后一个语句外，每一个语句最后都加上了逗号（**不要用中文标点符号！**）。这是json文件的规范格式。

别着急，我们还需要为你的自定义物品添加上中文名，否则当玩家选择中文作为游戏语言时，物品依然会显示注册名。

现在请在**assets/tutorial/lang**的文件目录下，也就是**en_us.json**文件的同级目录下新建一个文件，命名为**zh_cn.json**。作为mod的中文语言文件。

打开**zh_cn.json**文件，在文件的中括号中写入下面这段内容：

```
{
    "item.tutorial.custom_item":"自定义物品"
}
```

没错，冒号的前半段内容是和英语语言文件保持一致的（**一定要一致！**），后半段内容替换为中文即可。

现在我们已经完成了语言文件的写入。如果你想要你的自定义物品的名称在游戏内呈现其他颜色，你可以这样更改：

以红色为例，在英语语言文件中:

```
{
    "item.tutorial.custom_item":"§cCustom item"
}
```

没错，你只需要在后面加上`§c`即可。更多颜色符合表示可以查询Wiki。

### 添加模型文件

我们的自定义物品还需要加入模型文件，告诉Minecraft如何渲染这个物品。

找到**resources/assets/tutorial**目录，在这个目录下新建一个目录，命名为**models**（如果你发现你的目录没有被拆分，无法选中**resources/assets/tutorial**目录，请在任意一个地方新建**models**目录，右键，重构，移动软件包或目录，更改到正确的目录下即可）。现在**models**目录应该和你的**lang**目录处于同级。

现在请在**models**目录下再新建一个目录，命名为**item**，之后我们的所有物品模型文件都会存放在这。

在**item**目录下，新建一个文件，命名为**custom_item.json**，命名必须和你的物品注册名一致！在这个文件中写入下面这段内容：

```
{
  "parent": "minecraft:item/generated",
  "textures": {
    "layer0": "tutorial:item/custom_item"
  }
}
```

这段内容是在告诉Minecraft以普通物品的渲染作为该物品的父类，使用`tutorial:item/custom_item`这个目录下的贴图文件进行单层渲染。

现在我们已经成功写入了自定义物品的模型文件。此时进入游戏，你会发现你的自定义物品从紫黑色的方块变为了紫黑色的方片。

### 添加贴图文件

来到最后一步！使用你的绘图工具为你的自定义物品绘制贴图。如果你不希望你的物品在Minecraft世界中显得很违和，那么请使用16x16的画布进行绘制，这是所有Minecraft物品的统一贴图大小。当然，贴图的大小并没有限制，你想要多精细的贴图都可以，只需要保证最后导出的格式是`.png`即可！

让我们回到**resources/assets/tutorial**目录，在这个目录下新建一个目录，命名为**textures**，在这个目录下再新建一个目录，命名为**item**，然后将你的`.png`格式的贴图文件命名为`custom_item`，然后放在**item**目录下。

大功告成！现在进入游戏，输入`/give @a tutorial:custom_item`指令获取你的自定义物品，你应该可以得到一个：**带有正确的名称，正常的模型，正常的贴图**的物品。如果你发现它和你预期的不一样，请回到之前几个步骤，检查是否所有的命名都正确，是否所有的格式都符合，是否所有的文件都在正确的目录下。如果你在指令中根本找不到`tutorial:custom_item`这个物品，说明你的物品并没有成功被注册到Minecraft中，请回到注册物品阶段进行检查。不要灰心！debug是开发任何项目时都不可避免的事。

## 将你的物品添加到创造物品栏中

每次都要通过指令来获取我们的自定义物品实在是太麻烦了，为了解决这个问题，我们可以为mod创建自己的创造物品栏列表。

现在请在register文件夹下新建一个Java类，命名为ModCreativeTabs。

就像我们在这份文档开头提到过的，NeoForge在注册任何自定义内容时都需要一个**延迟注册器**，在此处也不例外。接下来请在**ModCreativeTabs.java**中写入以下内容：

```
public class ModCreativeTabs {

    public static final DeferredRegister<CreativeModeTab> CREATIVE_TABS = DeferredRegister.create(Registries.CREATIVE_MODE_TAB, Tutorial.MODID);


}
```

这样，我们就为接下来的注册内容创建好了延迟注册器。接下来，写入以下代码来注册真正的自定义创造物品栏：

```

public class ModCreativeTabs {

    public static final DeferredRegister<CreativeModeTab> CREATIVE_TABS = DeferredRegister.create(Registries.CREATIVE_MODE_TAB, Tutorial.MODID);

    public static final Supplier<CreativeModeTab> TUTORIAL_CUSTOM_CREATIVE_TAB = CREATIVE_TABS.register("tutorial_custom_creative_tab", () ->
            CreativeModeTab.builder()
                    .title(Component.translatable("tutorial_custom_creative_tab"))
                    .icon(() -> new ItemStack(ModItems.CUSTOM_ITEM.get()))
                    .displayItems(
                        (parameters, output) -> {
                        output.accept(ModItems.CUSTOM_ITEM_ONE.get());
                        output.accept(ModItems.CUSTOM_ITEM_TWO.get());
                        output.accept(ModItems.CUSTOM_ITEM_THERR.get());
                        output.accept(ModItems.CUSTOM_ITEM_FOUR.get());
                        }
                    )
                    .build());

}

```

这段代码看起来很复杂，但是不用担心，我会依次解答每部分的实际作用。

`public static final Supplier<CreativeModeTab> TUTORIAL_CUSTOM_CREATIVE_TAB = CREATIVE_TABS.register()`这段语句与我们在注册自定义物品时是一样的，都是使用了延迟注册方法通过一个延迟注册器总线注册一个新的自定义内容。

在`register()`中，我们填入了两个参数，前一个`tutorial_custom_creative_tab`是这个自定义创造物品栏的注册名，后一个则是它的属性设置。

在设置自定义创造物品栏的属性时，我们调用了一个构造器：`CreativeModeTab.builder()`，然后开始构造：

- **.title()**：这条属性设置了该自定义创造物品栏的名称，其中我们使用了`Component.translatable()`，这个方法可以从mod的语言json文件中提取对应的元素，从而实现了在玩家选择不同语言文件时，鼠标悬停在创造物品栏上可以显示不同语言的翻译内容。json文件中的具体内容我们会在下面补上。
- **.icon()**：这条属性设置了该自定义创造物品栏的图标。还记得原版的每一个创造物品栏都有一个自己的图标吗？我们也需要为自定义创造物品栏选择一个图标。如果你想用mod的自定义物品作为图标，和上面的示例代码一样，调用自定义物品的代码名称即可。如果你想使用原版Minecraft中的物品作为图标，例如苹果：`Items.APPLE`。**注意！**，使用原版物品时就不用在最后加上`.get()`方法的调用了。
- **.displayItems()**：这条属性设置了该自定义创造物品栏中需要展示哪些物品。**(parameters, output) -> {}**是lambda表达式的固定写法，后面的中括号中可填入你想要展示的物品，展示顺序与代码填入顺序有关。

最后，创建一个注册方法：

```

public class ModCreativeTabs {
    public static final DeferredRegister<CreativeModeTab> CREATIVE_TABS = DeferredRegister.create(Registries.CREATIVE_MODE_TAB, Tutorial.MODID);

    public static final Supplier<CreativeModeTab> TUTORIAL_CUSTOM_CREATIVE_TAB = CREATIVE_TABS.register("tutorial_custom_creative_tab", () ->
            CreativeModeTab.builder()
                    .title(Component.translatable("tutorial_custom_creative_tab"))
                    .icon(() -> new ItemStack(ModItems.CUSTOM_ITEM.get()))
                    .displayItems(
                            (parameters, output) -> {
                                output.accept(ModItems.CUSTOM_ITEM_ONE.get());
                                output.accept(ModItems.CUSTOM_ITEM_TWO.get());
                                output.accept(ModItems.CUSTOM_ITEM_THERR.get());
                                output.accept(ModItems.CUSTOM_ITEM_FOUR.get());
                            }
                    )
                    .build());
    
    public static void register(IEventBus eventBus) {
        CREATIVE_TABS.register(eventBus);
    }
}

```

现在我们已经注册好了自定义创造物品栏,接下来和自定义物品一样，我们还需要在语言json文件中加上它的翻译内容，然后将它注册到主类的mod总线中。

打开**resources/assets/tutorial/lang/**路径下的**en_us.json**文件，在文件中加入这样的内容：

```
{
    ...

    "itemGroup.tutorial_custom_creative_tab":"Custom Creative Tab"

    ...
}
```

然后打开**zh_cn.json**文件，加上：

```
{
    ...

    "itemGroup.tutorial_custom_creative_tab":"自定义创造物品栏"

    ...
}
```

这样我们就做好了自定义创造物品栏的中英双语文本准备。

现在打开Tutoial.java文件，在主类方法`public Tutorial(IEventBus modEventBus, ModContainer modContainer) {}`中加上：

```
public Tutorial(IEventBus modEventBus, ModContainer modContainer) {

        modEventBus.addListener(this::commonSetup);

        NeoForge.EVENT_BUS.register(this);

        ModItems.ITEMS.register(modEventBus);

        // 注意！ModCreativeTabs的注册一定要在ModItems的注册之后！否则会出现依赖错误导致游戏启动时报错
        ModCreativeTabs.CREATIVE_TABS.register(modEventBus);

        modEventBus.addListener(this::addCreative);

        modContainer.registerConfig(ModConfig.Type.COMMON, Config.SPEC);
    }
```

一切准备就绪，现在启动游戏，进入你的存档并更改为创造模式。打开背包，点击翻页，你应该会看到你的自定义创造物品栏，其中展示着你填入的物品。