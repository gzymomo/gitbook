## DevOps

[DevOps](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247488442&idx=2&sn=4bd13ebf1975dd75d79929272e5e6711&chksm=e91b76a6de6cffb0a2d68ae9a8469ac19a85bdf5e8be9bf945ac8347651ebbc6a5108ca20a73&scene=21#wechat_redirect)是Development和Operations的组合，是一种方法论，是一组过程、方法与系统的统称，用于促进应用开发、应用运维和质量保障（QA）部门之间的沟通、协作与整合。以期打破传统开发和运营之间的壁垒和鸿沟。![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPp3VTuibaiaCibduicia9btH8Ecr7GYXOAE335LciaeF3D6BtHQo5nQH2VWTcSM9VSdicQAABljrib6DicUPWQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)[DevOps](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247491328&idx=1&sn=ac1def9c84e1c2a2bad4bd7eed2f2974&chksm=e91b7a1cde6cf30aafe836e6e3023e4032522c0e41bb4cce56620fd0d310c613cac164b94ea5&scene=21#wechat_redirect)是一种重视“软件开发人员（Dev）”和“IT运维技术人员（Ops）”之间沟通合作的文化、运动或惯例。通过自动化“软件交付”和“架构变更”的流程，来使得构建、测试、发布软件能够更加地快捷、频繁和可靠。具体来说，就是在软件交付和部署过程中提高沟通与协作的效率，旨在更快、更可靠的的发布更高质量的产品。

也就是说[DevOps](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247495238&idx=2&sn=b6a4378b649b194b6bc3ae7c83ff3e46&chksm=e9188b5ade6f024c6991860d12c103370b51580234f7a169cf2e5a96a4294f2e72fecd440c0b&scene=21#wechat_redirect)是一组过程和方法的统称，并不指代某一特定的软件工具或软件工具组合。各种工具软件或软件组合可以实现DevOps的概念方法。其本质是一整套的方法论，而不是指某种或某些工具集合，与软件开发中设计到的OOP、AOP、IOC（或DI）等类似，是一种理论或过程或方法的抽象或代称。

## CI

CI的英文名称是Continuous Integration，中文翻译为：[持续集成](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247491042&idx=3&sn=c5be7a416d0613b3ec89507d5f8e832d&chksm=e91b78fede6cf1e8dde69008cb14613035013ded01e8b99990030616344115922f072e41011c&scene=21#wechat_redirect)。![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPp3VTuibaiaCibduicia9btH8Ecr6GowxHmgT9Ower4TR1JAmNLpcw3RaMzbpZokHsq8TEgZiaGYIvREgkA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)CI中，开发人员将会频繁地向主干提交代码，这些新提交的代码在最终合并到主干前，需要经过编译和自动化测试流进行验证。

[持续集成（CI）](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247499493&idx=3&sn=4453fb452640045f0b7a2b020a4de61a&chksm=e9189bf9de6f12ef78a70fd575f1e76b377fd454539e94b9e869270a99fb08f48278154be2fa&scene=21#wechat_redirect)是在源代码变更后自动检测、拉取、构建和（在大多数情况下）进行单元测试的过程。持续集成的目标是快速确保开发人员新提交的变更是好的，并且适合在代码库中进一步使用。CI的流程执行和理论实践让我们可以确定新代码和原有代码能否正确地集成在一起。

## CD

CD可对应多个英文名称，持续交付Continuous Delivery和持续部署Continuous Deployment，一下分别介绍。

查了一些资料，关于持续交互和持续部署的概念比较混乱，以下的概念总结按大部分的资料总结而来。

## 持续交付

完成 CI 中构建及单元测试和集成测试的自动化流程后，持续交付可自动将已验证的代码发布到存储库。为了实现高效的持续交付流程，务必要确保 CI 已内置于开发管道。持续交付的目标是拥有一个可随时部署到生产环境的代码库。![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPp3VTuibaiaCibduicia9btH8EcrTjRF1k5HDibeeK8C99W4Hk5jKXMcE9n4kYLTMO6llltcqMJfebicrekw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)在持续交付中，每个阶段（从代码更改的合并，到生产就绪型构建版本的交付）都涉及测试自动化和代码发布自动化。在流程结束时，运维团队可以快速、轻松地将应用部署到生产环境中或发布给最终使用的用户。

## 持续部署

对于一个成熟的CI/CD管道（Pipeline）来说，最后的阶段是持续部署。作为持续交付——自动将生产就绪型构建版本发布到代码存储库——的延伸，持续部署可以自动将应用发布到生产环境。![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPp3VTuibaiaCibduicia9btH8EcrHyZLYNKuSlD2VxHzbxniaqqSSNXS5x36uhnet4bxDE7KhxTI0gVKDpQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)持续部署意味着所有的变更都会被自动部署到生产环境中。持续交付意味着所有的变更都可以被部署到生产环境中，但是出于业务考虑，可以选择不部署。如果要实施持续部署，必须先实施持续交付。

持续交付并不是指软件每一个改动都要尽快部署到产品环境中，它指的是任何的代码修改都可以在任何时候实施部署。

持续交付表示的是一种能力，而持续部署表示的则一种方式。持续部署是持续交付的最高阶段。

## Agile Development

另外一个概念，也就是所谓的敏捷开发，似乎还没有所谓的简称，而且这个称呼似乎在国内被滥用了。敏捷开发着重于一种开发的思路，拥抱变化和快速迭代。如何实现敏捷开发，目前似乎尚没有完善的工具链，更多的是一种概念性，调侃的说法“既想马儿跑得快，又想马儿不吃草”的另外一种说法。![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPp3VTuibaiaCibduicia9btH8EcrwaicgJBicHz1wQCAahXhmKLTuYI8edp5icBibMcBgF0OdHd7cptXaPNiaJQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)上图揭示了敏捷开发的一些内涵和目标，似乎有点儿一本真经的胡说八道的意思。

## CI、CD、DevOps关系

概念性的内容，每个人的理解都有所不同。就好比CGI 这个词，即可以理解成CGI这种协议，也可以理解成实现了CGI协议的软件工具，都没有问题，咬文嚼字过犹不及。留下一图：![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPp3VTuibaiaCibduicia9btH8Ecraic4WZwqzDwbxrOLq4q3sdxbQCchTbgXx3TWxqEP5RjBClZUfBRjtibw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)