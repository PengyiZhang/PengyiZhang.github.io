# 【深度学习技术】如何用C++加载pytorch模型并进行推理部署 (三)

## "将C++ tensor图像拷贝至opencv Mat中"

如何高效地从pytorch C++ tensor结构中将超分输出的图像拷贝至opencv用于后续处理，具体代码如下：

    torch::Tensor out_tensor = module->forward(inputs).toTensor();
    assert(out_tensor.device().type() == torch::kCUDA);
    out_tensor=out_tensor.squeeze().detach().permute({1,2,0});
    out_tensor=out_tensor.mul(255).clamp(0,255).to(torch::kU8);
    out_tensor=out_tensor.to(torch::kCPU);
    cv::Mat resultImg(512, 512,CV_8UC3);
    std::memcpy((void*)resultImg.data,out_tensor.data_ptr(),sizeof(torch::kU8)*out_tensor.numel();

----

2020-03-19