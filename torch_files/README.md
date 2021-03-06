## pytorch cononical form

```
train_op = tf.train.AdamOptimizer(learning_rate=0.01).minimize(loss)   # tensorflow

net = Net()
optimizer = optim.Adam(net.parameters(), lr = 0.01)
loss_fn = nn.MSELoss()

while True:
    optimizer.zero_grad()
    y = net(x)
    loss = loss_fn(y,target)
    loss.backward()
    optimizer.step()

```



## pytorch tips
* https://pytorch.org/tutorials/
* device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
*  save & restore
```
torch.save(model.state_dict(), os.path.join(model_dir, 'epoch-{}.pth'.format(epoch)))

model.load_state_dict(torch.load('xxx.pth', map_location = device))
```
* network weights copy
```
net1.load_state_dict(net2.state_dict())
```
* PyTorch에서는 모델을 저장할 때 .pt 또는 .pth 확장자를 사용하는 것이 일반적인 규칙입니다.  ---> pt, pth는 차이 없고, 선택의 문제임.

* gradient cliping: https://pytorch.org/docs/stable/nn.html#clip-grad-norm
```
torch.nn.utils.clip_grad_norm_(model.parameters(), clip_value)
torch.nn.utils.clip_grad_value_(parameters, clip_value)
```
* learning rate 변경
```
def adjust_learning_rate(optimizer, lr):
    for param_group in optimizer.param_groups:
        param_group['lr'] = lr
```
* torch.gather
<p align="center"><img src="asset/pytorch_gather.png" />  </p>

* Attention Mask: http://juditacs.github.io/2018/12/27/masked-attention.html




## pytorch image load
```
import torchvision.models as models
import torchvision.transforms as transforms
import os
class MyVGG(nn.Module):
    def __init__(self,original_model):
        super(MyVGG, self).__init__()
        # vgg16에는 features, avgpool, classifier로 나누어져 있다.
        self.features = nn.Sequential(
            # features에서 마지막 3번재
            *list(original_model.features.children())[:-3]
        )
    def forward(self, x):
        x = self.features(x)
        return x 
os.environ['TORCH_HOME'] = './pretrained'
vgg16 = models.vgg16(pretrained=True, progress=True)  # 528M

transform=transforms.Compose([ transforms.ToTensor(),transforms.Normalize(mean=[0.485, 0.456, 0.406],std=[0.229, 0.224, 0.225])])
img = Image.open('dog.jpg')  # (576, 768, 3)
img = transform(img)  # torch.Size([3, 576, 768])

imgs = torch.unsqueeze(img,0)

feature1 = vgg16(imgs)  # ---> (N,1000)

extractor = MyVGG(vgg16)

featres2 = extractor(imgs)

```



## pytorch 기본 연산
import torch.nn.functional as F

F.linear(input, weight, bias=None)  y=xA^T + b
Input: (N, *, in_features) where * means any number of additional dimensions
Weight: (out_features, in_features)(out_features,in_features)
```
x = torch.randn(2,3)
w = torch.randn(3,4)
b = torch.randn(4)
x.mm(w) +b  # x.matmul(w)+b = torch.mm(x,w)+b = torch.nn.functional.linear(x,w.T,b)   ---> dot(벡터,벡터), mv(행렬,벡터), mm(행렬,행렬) ---> matmul이 모두 커버.
```

batch matrix multiplication
```
toch.bmm(A,B) # A(N,n,m) B:(N,m,k)  ---> (N,n,k)
```


