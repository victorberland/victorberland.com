<template>
  <div>
    <!-- <Header :siteTitle="siteTitle"/> -->
		<main>
			<slot></slot>
    </main>
    <!-- <Footer :siteTitle="siteTitle"/> -->
  </div>
</template>

<script>
import variables from 'saber/variables'
// import Header from './Header.vue'
// import Footer from './Footer.vue'

export default {
  components: {
    // Header,
    // Footer
  },

  props: ['page'],

  head() {
    const { excerpt } = this.page
    const { title, layout } = this.page
    let { description } = this.$siteConfig
    if (layout === 'page' || layout === 'post') {
      if (excerpt) {
        description = excerpt.replace(/<(?:.|\n)*?>/gm, '')
      }
    }
    return {
      title: title ? `${title} - ${this.siteTitle}` : this.siteTitle,
      meta: [
        description && {
          name: 'description',
          content: description
        }
      ].filter(Boolean),
      link: this.$feed
        ? [
            {
              rel: 'alternate',
              title: `${this.siteTitle} - Feed`,
              type: `application/${
                this.$feed.type === 'atom'
                  ? 'atom+xml'
                  : this.$feed.type === 'rss'
                  ? 'rss+xml'
                  : 'json'
              }`,
              href: this.$feed.permalink
            }
          ].filter(Boolean)
        : []
    }
  },

  computed: {
    siteTitle() {
      return this.$siteConfig.title || 'Your Awesome Title'
    }
  },
	mounted() {
		(function(f, a, t, h, o, m){
			a[h]=a[h]||function(){
				(a[h].q=a[h].q||[]).push(arguments)
			};
			o=f.createElement('script'),
			m=f.getElementsByTagName('script')[0];
			o.async=1; o.src=t; o.id='fathom-script';
			m.parentNode.insertBefore(o,m)
		})(document, window, '//a.v4.is/tracker.js', 'fathom');
		fathom('set', 'siteId', 'ABEUX');
		fathom('trackPageview');
	}
}
</script>

<style lang="scss">
@import url(https://brick.freetls.fastly.net/Montserrat:400,600);
body {
	margin: 0;
	background: #0D0D0D;
	font-family: 'Montserrat', 'Avenir', 'Helvetica Neue', sans-serif;
}
.post {
	margin: 150px;
	max-width: 900px;
	color: #ffffff;
	.post-title {
		font-size: 18px;
		text-transform: uppercase;
		letter-spacing: 1px;
		font-weight: bold;
		line-height: 180%;
		margin-bottom: 40px;
	}
	.post-content {
		margin-top: 60px;
		line-height: 220%;
		font-size: 17px;
		letter-spacing: 0.2px;
		font-weight: 400;
		a {
			$link-color: rgba(255,255,255,0.2);
			color: white;
			background: linear-gradient(0deg, $link-color 50%, transparent 50%);
			opacity: 0.8;
			padding: 5px 5px;
			text-decoration: none;
			transition: 0.2s ease;
			border-bottom: 0px solid $link-color;
			&:hover {
				border-bottom: 2px solid $link-color;
				opacity: 1;
			}
		}
	}
}
@media(max-width: 700px) {
	.post {
		margin: 50px;
		.post-content {
			font-size: 16px;
		}
	}
}
</style>
