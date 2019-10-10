[官网链接](https://jekyllrb.com/docs/installation/macos/)  
# 1 环境准备

#### 安装原因
由于Mac电脑自带的Ruby版本过低（2.3.x），而安装Jekyll要求Ruby版本>2.4.0  

#### 利用Homebrew安装Ruby
- 首先需要安装Homebrew 
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
- 再安装Ruby

```
brew install ruby
```
- 然后导入Ruby的环境变量
```
#注意利用export导入这种方式是临时环境变量，重新开一个终端导入的这个环境变量就不存在了  
export PATH=/usr/local/opt/ruby/bin:$PATH
```
- 检查Ruby版本

```
ruby -v
```
- 安装Jekyll并导入Jekyll环境变量

```
gem install --user-install bundler jekyll  

export PATH=$HOME/.gem/ruby/X.X.0/bin:$PATH  
#用安装的Ruby版本号的前两位替换 X.X
```

### 注意点

1. Every time you update Ruby to a version with a different first two digits, you will need to update your path to match.
2. 由于利用export导入的环境变量是临时变量，所以虽然电脑已经安装了最新版本的Ruby，但是每次使用Ruby是必须重新利用export导入。
3. 总之重新打开终端之后必须依次执行以下两行才能识别到Ruby和Jekyll

```
export PATH=/usr/local/opt/ruby/bin:$PATH  
export PATH=$HOME/.gem/ruby/X.X.0/bin:$PATH  
```  


# 2 主题安装与配置
### 下载主题并配置相关参数
##### 自己选择一个喜欢的主题，我选择的是scribble（[在GitHub上可以找到](https://github.com/muan/scribble)，然后安装它的说明下载、配置）  


---
##### Get started
1. Fork the repository

2. Clone the repository: git clone https://github.com/username/scribble

3. Run bundle install

4. Run Jekyll: bundle exec jekyll serve -w

5. Go to http://localhost:4000 for your site.

##### Make it yours  
1. Edit _config.yml, adn then rerun jekyll serve -w

2. Change about.md for blog intro

---
# 3 域名配置与发布  
和Windows类似，[可以参见另一篇文章](http://note.youdao.com/noteshare?id=eeef5c2683c4fa320d9e5439d9b4f460)。







