# frozen_string_literal: true

# 国内镜像源，提升依赖下载速度
source "https://gems.ruby-china.com"

# 锁定 Chirpy 主题稳定版本
gem "jekyll-theme-chirpy", "~> 7.4", ">= 7.4.1"

# HTML 测试工具，仅在 test 环境加载
gem "html-proofer", "~> 5.0", group: :test

# Windows 平台专属时区依赖（消除弃用警告）
platforms :windows, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# Windows 平台文件监听依赖（消除弃用警告）
gem "wdm", "~> 0.2.0", :platforms => [:windows]
