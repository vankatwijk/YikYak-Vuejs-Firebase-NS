<template>
    <Page>
        <ActionBar title="YikYak" android:flat="true"/>
        <TabView androidTabsPosition="bottom"
                 android:tabBackgroundColor="#53ba82"
                 android:tabTextColor="#c4ffdf"
                 android:selectedTabTextColor="#ffffff"
                 androidSelectedTabHighlightColor="#ffffff">
            <TabViewItem title="Feed">
                <GridLayout columns="*" rows="*">
                    <List/>
                </GridLayout>
            </TabViewItem>
            <TabViewItem title="Add Post">
                <GridLayout columns="*" rows="*">
                    <Post/>
                </GridLayout>
            </TabViewItem>
            <TabViewItem title="Feathers">
                <GridLayout columns="*" rows="*">
                    <Label class="message" text="Tab 3 Content" col="0" row="0"/>
                </GridLayout>
            </TabViewItem>
        </TabView>
    </Page>
</template>

<script >

  import Post from './Post'
  import List from './List'

  export default {
    components:{
        Post,
        List
    },
    data() {
      return {
      }
    }
  }
</script>

<style scoped>
    ActionBar {
        background-color: #53ba82;
        color: #ffffff;
    }

    .message {
        vertical-align: center;
        text-align: center;
        font-size: 20;
        color: #333333;
    }
</style>
