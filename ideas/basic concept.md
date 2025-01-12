# VR领域中的分支：MR与DR核心技术与概念分析
作为个人的思路整理文档，所有描述一切从简

## 一、目前的DR技术概述
DR（Diminished reality）被称为缩减现实，DR技术是从用户的视觉环境中移除或遮盖某些物体或信息，以实现“减少”现实内容的效果
<br>其核心目标是实时修改现实场景，使某些物体对用户不可见

### （一）DR的主要技术原理

#### 1. 图像/视频采集与处理
- 使用摄像头或传感器捕捉真实世界的场景
- 检测用户视野中的对象，并对其及进行标记和分割（通常采用目标检测和语义分割技术）
>标记和分割这一过程目前主要采用的是何种技术手段？是否可以用最新的SAM进行优化？

#### 2. 对象移除
- 利用计算机视觉技术对目标物体进行识别和分离
- 使用纹理补全（Texture Inpainting）或背景重建技术填补移除物体后的空白区域（涉及到掩模模型概念，后文细说）
>大阪大学的那篇论文其实也是一种背景重建技术，不过并不是真的重建了模型，只是生成了带有贴图的六面体掩模；我自己在课上想的方法应该可以归类到背景重建技术部分

#### 3. 场景重建
- 通过深度学习和计算摄影学，重建被移除物体后面的背景，并保证新生成场景的逼真和连续性

#### 4. 实时渲染
- 借助GPU和边缘计算，快速处理场景变化，保证DR在用户视角中的实时性和沉浸感

### （二）目前DR的一些实现方式

#### 1. photoAR+DR：大阪大学论文中索引[37]提及，缺点：实现需要十几个小时乃至几天
>具体技术原理和操作方式以后再研究
#### 2. RGB-D相机技术：大阪大学论文中索引[38]提及，缺点：需要大量点云数据，体积大延迟高，因此反而需要清除部分点云，但是这样又会导致生成的结果不准确
>看上去像是点云重建空间技术来实现DR后的背景补全，以后再研究

### （三）涉及到DR的一些基本概念

#### 1. RGB-D相机：核心在于具备深度传感器
- 深度传感器的基本工作原理有如下几种类型
  - 结构光：通过投射条纹或点阵，然后摄像头捕捉到变形模式来计算距离
  - 时间飞行：通过发送红外光脉冲并测量光脉冲反射回来的时间计算深度信息（SLAM是不是用这种原理实现的？）
  - 立体视觉：通过两个摄像头从不同角度拍摄同一个场景，计算视差来推测物体深度

- 深度图特点：为每一个像素提供一个距离值，表示该像素到相机的距离

- 涉及到我们研究方面的主要应用：AR、物体识别与跟踪、生成3D模型
    >其中物体识别和跟踪部分，用来进行物体检测、分类和跟踪，深度信息让计算机更好地理解物体的形状和位置，这里是不是可以利用SAM进行技术升级？

#### 2. 掩模模型（Mask model）
这里我想用我手绘的一组概念图来表达，我画到了草稿纸上，等以后有时间了再放到这篇文档里。简单来讲掩模就是掩盖了背景图的模型，在大阪那篇论文里的掩模其实是一个带有纹理贴图的六面体空间模型，然后企图用那个模型盖住目标白墙，即掩盖的含义

### （四）目前可以获取的正在研究DR技术的相关信息

#### 1. ZAUBAR公司提出的MR技术的研究方向
- 它里面提到了很多条目，但是我都忘了
- 等之后有时间在这里填上去……

#### 2. 一个网站但是我忘了叫什么，总之里面提到了有关图像分割的技术应用，与DR的主要技术原理第一条对应，不过他们用的是比较早期的模型，所以我觉得SAM应该有应用前景

---
## 二、MR技术概述

### （一）MR应用的开发相关内容

#### 1. MRUK（Mixed Reality Utility Kit）
一个开发工具包，旨在简化和加速混合现实（MR）应用的开发过程，特别是针对微软的Hololens设备
- 空间映射与环境感知：帮助开发者创建可以与环境交互的虚拟物体，比如利用空间映射实现虚拟对象与现实环境的精准对齐
- 手势和语音识别：可以让用户通过简单的手势或语音自然地控制虚拟物体
- 定位和追踪：高精度的设备定位和追踪功能，确保虚拟物体在用户视野中的位置和运动与实际环境一致，这个功能通常与SLAM（Simultaneous Localization and Mapping）技术紧密结合，帮助设备实时构建环境地图并精确定位，后文详细说明
- 多用户协作：不必多言，此部分也涉及到SLAM
- 其他交互界面设计的组件和工具：例如有眼动追踪、手势控制、语音命令等

#### 2. SLAM（Simultaneous Localization and Mapping）
该项技术可以实现精确的空间定位和环境感知，SLAM技术用于构建环境地图并追踪设备在空间中的位置，利用这些信息来实现虚拟对象能够与现实环境对齐
- 空间映射：MR技术中的空间映射功能基本依赖于SLAM技术来构建三维环境模型，确保虚拟物体可以正确显示在现实空间中的合适位置（这里我的理解是构建出来的虚拟空间与现实空间对齐的表现就是虚拟空间的边界、范围、碰撞体积等等参数与现实对齐）
- 定位和追踪：确保虚拟内容与现实世界保持一致（这里我的理解是当设备移动后，映射出来的空间是固定不动的，所以才会出现MR设备里显示的那样，即用户先带着设备扫描，等扫描完后确定就是这个空间啦，那么这个空间就不动了，之后只有在这个空间环境里的设备的移动）
>因为有这个定位功能在，所以可以定位在被映射好了的虚拟空间内的所有设备的移动

### （二）MR应用的思考

#### 1. 基本逻辑
- 数字孪生<br>
有一个现实空间，基于这个现实空间生成了一个对应的虚拟空间，到此为止类似于数字孪生的概念，即参考现实复刻一模一样的一个虚拟空间出来，那么我可以在这个虚拟空间内进行操作来模拟现实中发生同等操作后会出现的情况，这种想法的诞生之初源自建筑行业对现实改造成本巨大的一种优化思路，降低了设计失误的风险，也即在虚拟世界中模拟一遍建成后的效果以此来评估是否这份设计具备可行性或者足够好

  在这种情况下，实际上虽然可以更加精准把控设计后的实际效果，但是设计过程本身的成本提高了，因为必须要涉及到BIM模型的构建等等，因此数字孪生本质上并不是特别适合于方案的快速迭代过程，相比之下其更加适合于：（1）在已经明确了设计方案后的施工模拟环节；（2）对已经建成的但是需要长期维护并且维护成本很高的建筑的监测环节
  >实际上这种应用场景天然应该由政府去把控，为何？因为对于政府而言其有义务对建筑的整个生命周期进行监管，我这里的意思是：在设计方案确定后，政府委派的施工单位要经过严格的计算和施工，需要有BIM模型的支持；在不断的施工过程中这份BIM模型会趋于完善，当建筑建成后一个与现实房屋对应的BIM数据就被保管了起来

  >（1）从独立的个人空间而言：对于业主自身而言其理应有自由地支配房屋内部绝大部分装修的权力，因此对于政府而言其存储的对应的BIM数据只能是包含了其对应的现实空间的最基本的空间结构和样式，而如果这件事情可以做到，那么对于之后其要进行较大的装修工程而言就方便的多，因为接到客户委托的设计师可以根据自身的资格或者权限调用政府存储的BIM数据，有了该房屋的BIN数据，之后的设计和施工就更容易把控，当施工完成后设计团队再将新的BIM模型传输给政府进行数据更新，因为对于政府而言也是更新了政府内部的数据信息，因而政府也应该给设计团队一部分额外的报偿
  >
  >（2）从范围更大的片区空间而言：一片小区、一整个建筑区块乃至一整片城区结构的BIM数据如果都被存储起来，那么对于政府对城区进行宏观的道路规划、城区容貌更加便于管理。当某块待建基地明确由某团队承包，那么该团队（包含后期施工团队）需要将自己负责的区块范围的BIM数据传输给政府进行数据更新
- MR应用<br>
MR就是简化版的数字孪生，其核心思路在于建立一个与现实空间对应的虚拟空间，通过摄像头可以捕捉现实的空间数据并进行虚拟空间的构建，最后用户所看到的现实空间，但是背后存在着一个被构建了的虚拟空间，这个虚拟空间的作用就是作为虚拟物体发生实际交互的场所，比如碰撞体积。那么用户在使用MR应用的时候就会感觉是虚拟的物体与真实的世界发生了交互，但实际上是虚拟的物体与一个基于现实空间的所构建的虚拟空间的交互

#### 2. 局限性
如果是严格按照“混合现实”的概念，那么所谓混合就应该包括增加和删除等更为复杂的交互逻辑，但目前所谓的混合实际上只能做到：（1）与现实世界的物理碰撞体积进行映射；（2）在现实世界的基础上增加虚拟物体，但不能通过虚拟的手段去修改用户所捕捉到的现实的空间环境，因此MR技术要进行一次大的升级就必须要对DR技术进行突破

---
## 三、大阪大学论文批判（附带）
先用无结构文字编辑，再去梳理层级逻辑<br>
本篇论文主要做的一件事就是对DR技术的一次尝试，文中主要是对一面墙壁进行DR操作，后面被DR技术所操作而隐藏或是遮盖掉的部分就称其为DR对象。论文中面对的问题是：要把一面墙作为DR对象，然后显露出墙背后的实际内容。一开始我以为设备仅仅是头显，但是后来精读发现实际上还是在墙背后的那个房间里的中心位置放了一个全景摄像头，实际上是根据这个全景摄像头捕捉的全景图像去生成最后要覆盖掉背景墙的掩模模型，其基本思路是：获取了这个全景图像以后，通过某种技术手段将这个全景图投射到一个六面体模型中，就是把一个六面体的六个面分别添加一个纹理贴图，而这个平面贴图来源于一个球形的全景图，而且是贴到了六面体的内部，那么这个贴图是单向的，从贴图背面看过去就是透明的，那么就可以透过其中一面看到其余五个被贴上了纹理的面。
<br>至于文中提到的GAN网络我看下来好像是仅仅是为了插入一个“用户可以用草图生成渲染图”这么一种功能，基于此去训练的一个神经网络，但是这一部分本身并没有什么创新，或者说我直接用一个现有的图像生成AI模型不可以吗？为什么还要自己再单独去进行一个GAN的神经网络的训练？此外好像这个神经网络就没有再被应用到的地方了？至少好像对于DR对象这一过程本身没有起到什么核心作用，所以看下来感觉像是强行塞进去的部分，也就是说在本文中的DR技术本身并没有跟神经网络扯上关联；神经网络是现成的，只需要输入对应的训练集即可，全景图投影到六面体技术好像也是现成的，甚至因为放置在被挡住的房间的中心的摄像头因为放在的是房间的桌子上，导致最后投影到六面体底面的贴图甚至是平面的效果，或者说畸变很严重的效果，完全与空间内的三维物体效果不一样，这种差异是不可接受的。
<br>最后文中很简短地提到了一个局限性就是识别的物体边缘的效果不好，因为对于用户要观测DR对象，在这个DR对象前面会有许多遮挡物件，所以需要把这些遮挡物件识别后保留，让他们不要被掩模模型所遮盖，然后文中并没有明说这种识别技术他们用的是哪种，总之最后边缘效果不好。虽然文中仅把其作为一个不足之处描述，但是我觉得这个部分反而有可能作为一个核心的议题去讨论，因为针对于物体识别这个技术，也许可以基于此换一个新的实现DR技术的思路
<br>补充，文中还提到了关于使用BIM技术去优化整个系统，但这里一个问题就是，如果是针对一般用户而言，是不具备BIM模型的制作条件的，但是如果说一个用户最后决定了就是要装修的话，那么设计团队当然是最终一定会有设计重建室内模型的这么一个工作阶段，这个没问题；但是如果是一个只是想带着设备去尝试着修改室内情况而并没有决定了一定要进行装修的话，那么BIM模型的引用就不现实了，或者换言之，既然都已经有一个BIM模型了，为什么不直接把这个模型作为VR应用中的可以让用户体验的场景，还需要用上DR或者MR？简而言之，DR或者MR的本质都是要基于现实的操作，如果连现实都没有了，变成了纯粹的VR，那么就没有必要考虑DR和MR了，想要哪个物体消失就直接消失好了

## 四、我个人对DR技术的想法

### （一）目前解决思路的考察

#### 1.基于前文提到的DR技术的两个主要方向，一个是内部纹理修复，一个是背景重建技术
- 内部纹理修复<br>
内部纹理修复就是给一个身处某个背景的物体表面上贴上一个周围环境的纹理，原理类似动物保护色，但这个技术局限性太大了，大多数情况无法适应背景复杂且动态变化的使用情景
- 背景重建技术<br>
类似大阪论文中的方法应该就是属于背景重建技术，重建了一个实时固定在某个具体位置的带有特定纹理贴图的虚拟六面体模型作为掩模模型盖住DR对象；好像大部分所谓的背景重建技术其实就是用一个掩模模型盖在DR对象上，所以解决的思路就是如何生成合理的这个掩模的表面贴图。
- 我的思路<br>
仅针对建筑室内设计装修，我的思路应该归类于背景重建技术，但不是聚焦在如何生成合理的掩模的表面贴图；因为既然是仅针对室内设计部分，那么可以把目标对象抽象为空间结构与室内物件两个内容，我设想的内部空间结构是不可改变的，这里的空间结构指代硬结构，类似于墙面、底面和顶面，那么我的思路可能就无法实现大阪论文中的所谓的透视效果，虽然我并不觉得那篇论文思路很好。然后我通过设备对室内空间进行识别后，虽然是识别出了要掩盖的对象，但是并不是想着把对象剔除去给里面补充贴图，而是识别了最基本的硬结构部分后将其全部用掩模去覆盖，或者说，识别了全部的室内物件后，将其拿出来不要被掩模遮盖，因为我的这个掩模不是要遮盖物件，而是遮盖空间结构，，那么基于这种思路，如果我想要把某个物体给遮盖了，本质就是取消掩模对其的遮盖，让这个物件在三个维度上分别被遮盖结构的掩模所遮盖，这个思路只能说是可以去考虑，因为随便一想就可以想到反例，比如一个背景全部是桌子的物件就没办法用这个识别墙体的思路去实现了，但是如果说欸，那我把桌子也用同样的方式去当作墙体一样操控，那实际上就相当于把扫描到的物体全部生成一个完整的“待使用的掩模”了，那种方式无异于直接重建室内模型，并不是我们所希望的。
