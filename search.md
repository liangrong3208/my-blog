---
layout: page
title: 搜索
permalink: /search/
---

<style>
.search-container {
  margin: 20px 0;
}

#search-input {
  width: 100%;
  max-width: 500px;
  padding: 12px 16px;
  font-size: 16px;
  border: 2px solid #e1e4e8;
  border-radius: 6px;
  outline: none;
  transition: border-color 0.2s;
}

#search-input:focus {
  border-color: #0366d6;
}

#search-results {
  margin-top: 20px;
}

.search-result {
  padding: 15px 0;
  border-bottom: 1px solid #e1e4e8;
}

.search-result:last-child {
  border-bottom: none;
}

.search-result a {
  color: #0366d6;
  text-decoration: none;
  font-size: 18px;
  font-weight: 500;
}

.search-result a:hover {
  text-decoration: underline;
}

.search-date {
  display: block;
  color: #666;
  font-size: 14px;
  margin-top: 5px;
}

.search-excerpt {
  color: #555;
  font-size: 14px;
  margin-top: 8px;
  line-height: 1.5;
}

.no-results {
  color: #666;
  font-style: italic;
  padding: 20px 0;
}

.search-status {
  color: #666;
  font-size: 14px;
  margin-bottom: 10px;
}
</style>

<div class="search-container">
  <input type="text" id="search-input" placeholder="输入关键词搜索文章..." autocomplete="off">
  <div id="search-status" class="search-status"></div>
  <div id="search-results"></div>
</div>

<script>
(function() {
  const input = document.getElementById('search-input');
  const results = document.getElementById('search-results');
  const status = document.getElementById('search-status');
  let posts = [];
  let loaded = false;

  // 加载搜索索引
  fetch('{{ "/search.json" | relative_url }}')
    .then(response => {
      if (!response.ok) {
        throw new Error('加载失败: ' + response.status);
      }
      return response.json();
    })
    .then(data => {
      posts = data;
      loaded = true;
      status.textContent = '已加载 ' + posts.length + ' 篇文章，输入关键词开始搜索';
    })
    .catch(error => {
      status.textContent = '加载失败: ' + error.message;
      console.error('搜索索引加载失败:', error);
    });

  // 搜索功能
  input.addEventListener('input', function() {
    const query = this.value.trim().toLowerCase();
    
    if (!loaded) {
      results.innerHTML = '<div class="no-results">正在加载数据...</div>';
      return;
    }
    
    if (query.length === 0) {
      results.innerHTML = '';
      return;
    }
    
    if (query.length < 2) {
      results.innerHTML = '<div class="no-results">请输入至少2个字符</div>';
      return;
    }

    const matches = posts.filter(post => {
      const titleMatch = post.title && post.title.toLowerCase().includes(query);
      const contentMatch = post.content && post.content.toLowerCase().includes(query);
      return titleMatch || contentMatch;
    });

    if (matches.length === 0) {
      results.innerHTML = '<div class="no-results">未找到匹配的文章</div>';
      return;
    }

    results.innerHTML = matches.map(post => {
      const excerpt = post.content ? post.content.substring(0, 200) + '...' : '';
      return `
        <div class="search-result">
          <a href="${post.url}">${post.title}</a>
          <span class="search-date">${post.date}</span>
          ${excerpt ? `<div class="search-excerpt">${excerpt}</div>` : ''}
        </div>
      `;
    }).join('');
  });

  // 支持回车键搜索
  input.addEventListener('keypress', function(e) {
    if (e.key === 'Enter') {
      e.preventDefault();
    }
  });
})();
</script>
