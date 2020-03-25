# 【深度学习技术】小样本医学影像的深度学习关键技术之深度模型的可解释性


The success of deep learning has been witnessed as a promising technique for computer-aided biomedical image analysis, due to end-to-end learning framework and availability of large-scale labelled samples. However, in many cases of biomedical image analysis, deep learning techniques suffer from the small sample learning (SSL) dilemma caused mainly by lack of annotations. To be more practical for biomedical image analysis, in this paper we survey the key SSL techniques that help relieve the suffering of deep learning by combining with the development of related techniques in computer vision applications. In order to accelerate the clinical usage of biomedical image analysis based on deep learning techniques, we intentionally expand this survey to include the explanation methods for deep models that are important to clinical decision making. We survey the key SSL techniques by dividing them into five categories: (1) explanation techniques, (2) weakly supervised learning techniques, (3) transfer learning techniques, (4) active learning techniques, and (5) miscellaneous techniques involving data augmentation, domain knowledge, traditional shallow methods and attention mechanism. These key techniques are expected to effectively support the application of deep learning in clinical biomedical image analysis, and furtherly improve the analysis performance, especially when large-scale annotated samples are not available.


## Demos

https://github.com/PengyiZhang/MIADeepSSL/tree/master/00_Interpretability

基于两种常用的深度模型可解释性方法：类激活图CAM和Grad-CAM，对在MURA数据集上训练的DenseNet169模型进行解释。


![img](https://github.com/PengyiZhang/MIADeepSSL/raw/master/00_Interpretability/images/image2.png)

![img](https://github.com/PengyiZhang/MIADeepSSL/raw/master/00_Interpretability/images/image2_cam.png)

![img](https://github.com/PengyiZhang/MIADeepSSL/raw/master/00_Interpretability/images/image2_grad_cam3.png)

![img](https://github.com/PengyiZhang/MIADeepSSL/raw/master/00_Interpretability/images/image2_grad_cam2.png)

![img](https://github.com/PengyiZhang/MIADeepSSL/raw/master/00_Interpretability/images/image2_grad_cam1.png)

![img](https://github.com/PengyiZhang/MIADeepSSL/raw/master/00_Interpretability/images/image2_grad_cam0.png)

CAM 基于GAP以及FC来对最后一个卷积层的输出构建可视化解释，而Grad-CAM则用梯度来代替卷积的输出来构建可视化解释，是CAM的一种推广。CAM只能在最后一个卷积层的输出上构建可视化解释，而Grad-CAM则可以对任意卷积的输出进行构建，因此，在本例程中，DenseNet169具有4个dense blocks，CAM只对第四个dense block进行了Visual CAM的构建，而Grad-CAM则对四个dense block的输出都进行了可视化显示。结果也表明，在CAM与Grad-CAM在最后一个卷积层的输出上得到的结果是一样的，而Grad-CAM则能够在前三个dense blocks也构建可视化解释。

通过Visual CAM以及Visual Grad-CAM来看，模型较高层的时候确实能够大致定位到图片中的语义位置，即病变发生的位置。这可以相比原始图像来讲，更加有助于医生做出最终诊断。而Visual Grad-CAM在较低层的解释结果可以看到，深度模型较低层的卷积输出结果含有较多的底层特征，较少的语义特征，加之分辨率比较高，如何能够将高层语义信息更好的回传至底层，对于进一步实现弱监督的目标定位以及语义分割都将有着非常重要的意义。


## code

    import re
    import torch
    import torch.nn as nn
    import torch.nn.functional as F
    import torch.utils.model_zoo as model_zoo
    from collections import OrderedDict


    class _DenseLayer(nn.Sequential):
        def __init__(self, num_input_features, growth_rate, bn_size, drop_rate):
            super(_DenseLayer, self).__init__()
            self.add_module('norm1', nn.BatchNorm2d(num_input_features)),
            self.add_module('relu1', nn.ReLU(inplace=True)),
            self.add_module('conv1', nn.Conv2d(num_input_features, bn_size *
                            growth_rate, kernel_size=1, stride=1, bias=False)),
            self.add_module('norm2', nn.BatchNorm2d(bn_size * growth_rate)),
            self.add_module('relu2', nn.ReLU(inplace=True)),
            self.add_module('conv2', nn.Conv2d(bn_size * growth_rate, growth_rate,
                            kernel_size=3, stride=1, padding=1, bias=False)),
            self.drop_rate = drop_rate

        def forward(self, x):
            new_features = super(_DenseLayer, self).forward(x)
            if self.drop_rate > 0:
                new_features = F.dropout(new_features, p=self.drop_rate, training=self.training)
            return torch.cat([x, new_features], 1)


    class _DenseBlock(nn.Sequential):
        def __init__(self, num_layers, num_input_features, bn_size, growth_rate, drop_rate):
            super(_DenseBlock, self).__init__()
            for i in range(num_layers):
                layer = _DenseLayer(num_input_features + i * growth_rate, growth_rate, bn_size, drop_rate)
                self.add_module('denselayer%d' % (i + 1), layer)


    class _Transition(nn.Sequential):
        def __init__(self, num_input_features, num_output_features):
            super(_Transition, self).__init__()
            self.add_module('norm', nn.BatchNorm2d(num_input_features))
            self.add_module('relu', nn.ReLU(inplace=True))
            self.add_module('conv', nn.Conv2d(num_input_features, num_output_features,
                                            kernel_size=1, stride=1, bias=False))
            self.add_module('pool', nn.AvgPool2d(kernel_size=2, stride=2))


    class DenseNet(nn.Module):
        r"""Densenet-BC model class, based on
        `"Densely Connected Convolutional Networks" <https://arxiv.org/pdf/1608.06993.pdf>`_

        Args:
            growth_rate (int) - how many filters to add each layer (`k` in paper)
            block_config (list of 4 ints) - how many layers in each pooling block
            num_init_features (int) - the number of filters to learn in the first convolution layer
            bn_size (int) - multiplicative factor for number of bottle neck layers
            (i.e. bn_size * k features in the bottleneck layer)
            drop_rate (float) - dropout rate after each dense layer
            num_classes (int) - number of classification classes
        """

        def __init__(self, growth_rate=32, block_config=(6, 12, 24, 16),
                    num_init_features=64, bn_size=4, drop_rate=0, num_classes=1000, input_shapes=(3, 224, 224)):

            super(DenseNet, self).__init__()
            self.num_classes = num_classes
            # First convolution
            self.features_first = nn.Sequential(OrderedDict([
                ('conv0', nn.Conv2d(input_shapes[0], num_init_features, kernel_size=7, stride=2, padding=3, bias=False)),
                ('norm0', nn.BatchNorm2d(num_init_features)),
                ('relu0', nn.ReLU(inplace=True)),
                ('pool0', nn.MaxPool2d(kernel_size=3, stride=2, padding=1)),
            ]))

            # Each denseblock
            num_features = num_init_features

            self.features = nn.ModuleList()

            for i, num_layers in enumerate(block_config):
                _denseblock = nn.Sequential()

                block = _DenseBlock(num_layers=num_layers, num_input_features=num_features,
                                    bn_size=bn_size, growth_rate=growth_rate, drop_rate=drop_rate)
                _denseblock.add_module('denseblock%d' % (i + 1), block)
                num_features = num_features + num_layers * growth_rate
                if i != len(block_config) - 1:
                    trans = _Transition(num_input_features=num_features, num_output_features=num_features // 2)
                    _denseblock.add_module('transition%d' % (i + 1), trans)
                    num_features = num_features // 2
                
                else:
                    # Final batch norm
                    _denseblock.add_module('norm5', nn.BatchNorm2d(num_features))
                
                self.features.append(_denseblock)
                



            # Linear layer

            self.classifier = nn.Linear(num_features, num_classes)

            # Official init from torch repo.
            for m in self.modules():
                if isinstance(m, nn.Conv2d):
                    nn.init.kaiming_normal_(m.weight)
                elif isinstance(m, nn.BatchNorm2d):
                    nn.init.constant_(m.weight, 1)
                    nn.init.constant_(m.bias, 0)
                elif isinstance(m, nn.Linear):
                    nn.init.constant_(m.bias, 0)

        def save_gradient(self, grad):
            alpha_c = grad.mean(dim=(-2,-1)) # mean over space slice, batch_size x channels
            self.alpha.append(alpha_c.unsqueeze(-1))  

        def clear_gradient(self):
            """
            清除gradient
            """
            self.alpha = []

        # 注册梯度
        def sethook(self):
            for dense_feature in self.dense_features:
                dense_feature.register_hook(self.save_gradient)

        def get_cam(self):
            features = F.relu(self.dense_features[-1])
            batch_size, channels, height, width = features.shape
            
            features = features.view((batch_size, channels, height*width)).transpose(1,2).view((batch_size*height*width, channels))
            cam = self.classifier(features).transpose(0,1).view((batch_size, self.num_classes, height, width))-self.classifier.bias
            return cam

        def get_gradcam(self, cls_score):
            grad_cam_blocks = []

            mean_class_y = cls_score.sum(dim=0) # classes_num

            for y in mean_class_y: # 单个class梯度回传
                self.clear_gradient() # 清空存储梯度
                grad_cam_block = [] # 存储该class下的grad-cam
                self.sethook() # 注册梯度回调函数
                y.backward(retain_graph=True) # 梯度回传，获取梯度
                # 
                for b, alpha in enumerate(self.alpha, 1): # 回传顺序反过来了
                    grad_cam_block.append(F.relu(torch.sum(alpha.unsqueeze(-1)*self.dense_features[-b], dim=1, keepdim=True)))

                grad_cam_blocks.append(grad_cam_block)

                # 顺序与self.dense_features一样
                grad_cam_blocks = [torch.cat([grad_cam_blocks[c][b] for c in range(len(mean_class_y))], 1) for b in range(len(grad_cam_blocks[0])-1, -1, -1)]
            
            return grad_cam_blocks



        def forward(self, x, block=-1):

            self.dense_features = []

            x = self.features_first(x)
            # densenet classification branch
            for i, l in enumerate(self.features):
                x = l(x)
                self.dense_features.append(x)

            out = F.relu(x, inplace=True)

            out = F.adaptive_avg_pool2d(out, (1, 1)).view(x.size(0), -1)
            cls_score = F.sigmoid(self.classifier(out))

            cam = self.get_cam()

            grad_cams = self.get_gradcam(cls_score)
            

            return cls_score, cam, grad_cams


    def densenet169(pretrained=False, input_shapes=(3, 224, 224), **kwargs):
        r"""Densenet-169 model from
        `"Densely Connected Convolutional Networks" <https://arxiv.org/pdf/1608.06993.pdf>`_

        Args:
            pretrained (bool): If True, returns a model pre-trained on ImageNet
        """

        
        model = DenseNet(num_init_features=64, growth_rate=32, block_config=(6, 12, 32, 32), input_shapes=input_shapes,
                        **kwargs)
        # if pretrained:
        #     # '.'s are no longer allowed in module names, but pervious _DenseLayer
        #     # has keys 'norm.1', 'relu.1', 'conv.1', 'norm.2', 'relu.2', 'conv.2'.
        #     # They are also in the checkpoints in model_urls. This pattern is used
        #     # to find such keys.
        #     pattern = re.compile(
        #         r'^(.*denselayer\d+\.(?:norm|relu|conv))\.((?:[12])\.(?:weight|bias|running_mean|running_var))$')
        #     state_dict = model_zoo.load_url(model_urls['densenet169'])
        #     for key in list(state_dict.keys()):
        #         res = pattern.match(key)
        #         if res:
        #             new_key = res.group(1) + res.group(2)
        #             state_dict[new_key] = state_dict[key]
        #             del state_dict[key]
        #     model.load_state_dict(state_dict)
        return model



----

2020-03-25