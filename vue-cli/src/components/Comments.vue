<template>
  <div class="comments">
    <h2 class="comments__title">Comments</h2>

    <NewComment v-if="$store.state.user !== null" />

    <DisplayOptionsComments />

    <ul class="comments__list" v-if="sortedRootComments.length > 0">
      <Comment
        v-for="comment in sortedRootComments"
        :key="comment.id"
        :comment="comment"
        :comments="comments"
      />
    </ul>

    <div class="no-comments" v-if="sortedRootComments.length === 0">
      No comments so far
    </div>
  </div>
</template>

<script>
import NewComment from './NewComment'
import Comment from './Comment'
import DisplayOptionsComments from './DisplayOptionsComments'

import COMMENTS from '@/queries/Comments.gql'

export default {
  name: 'Comments',
  components: {
    NewComment,
    Comment,
    DisplayOptionsComments
  },
  props: ['itemId'],
  apollo: {
    comments: {
      query: COMMENTS,
      variables () {
        return { where: { item: { id: this.itemId } } }
      },
      update: ({ comments }) => comments
    }
  },
  computed: {
    sortedRootComments () {
      if (!this.comments) return []

      const comments = [...this.comments].filter(comment => !comment.parent)

      switch (this.$store.state.commentsOrder) {
        case 'newest':
          comments.sort((a, b) => {
            return new Date(b.created_at) - new Date(a.created_at)
          })
          break

        case 'oldest':
          comments.sort((a, b) => {
            return new Date(a.created_at) - new Date(b.created_at)
          })
          break

        case 'last_activity':
          comments.sort((a, b) => {
            let aCreatedAt = new Date(a.created_at)
            for (const comment of this.comments) {
              if (!comment.parent || comment.parent.id !== a.id) continue

              const date = new Date(comment.created_at)
              if (date > aCreatedAt) aCreatedAt = date
            }

            let bCreatedAt = new Date(b.created_at)
            for (const comment of this.comments) {
              if (!comment.parent || comment.parent.id !== b.id) continue

              const date = new Date(comment.created_at)
              if (date > bCreatedAt) bCreatedAt = date
            }

            return bCreatedAt - aCreatedAt
          })
          break
      }

      return comments
    }
  }
}
</script>

<style lang="scss">
@import '../variables.scss';

.comments {
  margin-top: 2rem;

  &__title {
    margin-top: 0;
    margin-bottom: 0;
  }

  &__list {
    list-style: none;
    margin-top: 0;
    margin-bottom: 0;
    padding-left: 0;

    > .comment {
      margin-top: 1rem;

      &:first-of-type {
        margin-top: 0;
      }

      > .comment__content {
        box-shadow: 2px 2px 10px 0 $shadow--light;
        border-radius: .5rem;
        border-left: none;

        .dark & {
          box-shadow: 2px 2px 10px 0 $shadow--dark;
        }
      }

      &--private {
        > .comment__content {
          overflow: hidden;

          position: relative;

          background: linear-gradient(180deg, rgba(255, 0, 0, .1) 0, rgba(255, 0, 0, 0) 7rem);

          &::before {
            content: '';

            position: absolute;
            left: 0;
            bottom: 0;

            width: 100%;
            height: 2px;

            background: $red;
          }
        }
      }
    }
  }
}

.no-comments {
  padding: 2rem;
  border-radius: .5rem;
  border: 1px solid #eee;
  color: #aaa;
  text-align: center;
  text-transform: uppercase;
  font-size: .9rem;
}
</style>
