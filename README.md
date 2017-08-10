基于 ``koa`` 和 ``react`` 搭建的个人博客

基于前后端分离的思想，后端提供接口，前端调用


## 技术栈

前端：
- react@15.3.1
- react-router@3.0.5
- redux@3.6.0
- webpack@1.13.2
- es6

后台：
- koa@2.0.0-alpha.8
- mongoose@4.11.1
- asyn/await


## 浏览器兼容

- Chrome
- Firefox
- Safari
- IE10+

## 简单介绍

异步fetch结合action的使用：
```

// 获取所有文章id、title
export const getArticles = () => {
  return dispatch => {
    const url = `${CONFIG.server}/api/getArts`
    return fetch(url, {
      method: 'POST',
      // 设置这个header，才能正确parse
      headers: { 'Content-Type': 'application/json' },
      mode: 'cors'
    })
      .then(response => response.json())
      .then(data => {
        console.log(data)
        dispatch({
          type: GET_ARTICLES,
          articles: data.articles
        })
      })
  }
}
```

```
// 获取所有文章action对应的reducer
case GET_ARTICLES:
    return Object.assign({}, defaultIssuesState, {
      isFetching: false,
      articles: action.articles
    })
```

异步dispatch（action）的处理
```
componentDidMount() {
    const { dispatch } = this.props
    NProgress.start()
    dispatch(getArticles())
      .then(() => {
        const { _id } = this.props.articles[0]
        dispatch(getArticleById(_id))
          .then(() => {
            NProgress.done()
            this.setState({
              article: this.props.article
            })
          })
      })
  }
```

jsonwebtoken在koa的实现
```
// token 验证
  router.post(
    '/valid',
    async(ctx, next) => {
      const { token } = ctx.request.body
      try {
        const decoded = jwt.verify(token, 'secret')
        // 过期
        if (decoded.exp <= Date.now()/1000 ) {
          ctx.body = {
            status: 0,
            msg: '登录状态已过期，请重新登录'
          }
          return
        }
        if (decoded) {
          // token is ok
          ctx.body = {
            status: 1,
            msg: '登陆验证成功'
          };
          return;
        }
      } catch(e) {
        if(e) {
          ctx.body = {
            status: 0,
            msg: e.message
          } 
        }
      }  
    }
  )

```

让nodejs支持es6，注意：nodejs主持async／await的话要8.0以上
```
require('babel-core/register')
require('babel-polyfill')
require('./app')
```
