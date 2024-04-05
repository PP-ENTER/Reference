# 모델 작성 버전1 내용입니다. (2024-04-04)

```python
############################################################
# accounts 

from django.contrib.auth.models import AbstractUser
from django.db import models


AbstractUser
# id: 1
# username: 엄영철
# firstname: 영철(null)
# lastname: 엄(null)

# --------

# user_id : test / password test1234! => "로그인 정보"
# nickname : zerochul

class CustomUser(AbstractUser):
    # user_no = models.AutoField(primary_key=True) # 1, 2, 3,.... # 자동 증가(serialNo)
    user_id = models.CharField(max_length=255, unique=True)  # test, leehojun ..    # 고유한 사용자 ID 필드 추가
    nickname = models.CharField(max_length=255) # 별명
    profile_image = models.ImageField(upload_to="users/images/", null=True, blank=True)
    # created_at = models.DateTimeField(auto_now_add=True) # join date
    is_active = models.BooleanField() # 접속여부 
    updated_at = models.DateTimeField(auto_now=True)


class Friend(models.Model): 
    user = models.ForeignKey(CustomUser, on_delete=models.CASCADE, related_name='friends')
    friend = models.ForeignKey(CustomUser, on_delete=models.CASCADE, related_name='friends_of')
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        unique_together = ('user', 'friend',)

    def __str__(self):
        return f"{self.user.username} - {self.friend.username}"
    

class FriendRequest(models.Model):
    from_user = models.ForeignKey(CustomUser, on_delete=models.CASCADE, related_name='sent_friend_requests')
    to_user = models.ForeignKey(CustomUser, on_delete=models.CASCADE, related_name='received_friend_requests')
    stauts = models.BooleanField() # 현재 상태 -> 1. 요청중, 거절, 2. 수락 => 상대방이 받으면 True -> Friend에 반영
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        unique_together = ('from_user', 'to_user')

############################################################
# facechats 
from django.conf import settings
from django.db import models

class FaceChat(models.Model):
    # face_chat_id = models.AutoField(primary_key=True) # id값은 기본적으로 자동으로 생성, 사용자 정의로 필드명을 선언할땐 이렇게 작성하고 이런 경우 자동으로 id는 생성되지x
    room_name = models.CharField(max_length=255)
    host_id = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name='hosted_chats')
    stauts = models.IntegerField()  # 0: 진행 중, 1: 종료, 2: 중지 -> boolena type 
    duration = models.DateTimeField()  # 날짜
    count = models.IntegerField() # 조회 수
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)


class FaceChatParticipant(models.Model):
    face_chat_id = models.ForeignKey(FaceChat, on_delete=models.CASCADE, related_name='participants')
    seqno = models.IntegerField() # 자동증가, 최대 4
    user_id = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name='participated_chats')
    joined_at = models.DateTimeField()

    class Meta:
        unique_together = ('face_chat', 'seqno')


class FaceChatTag(models.Model):
    face_chat_id = models.ForeignKey('FaceChat', on_delete=models.CASCADE, related_name='tags')
    tag = models.ForeignKey('posts.Tag', on_delete=models.CASCADE, related_name='facechats')

    class Meta:
        unique_together = ('facechat', 'tag')


############################################################
# notices
from django.contrib.auth.models import AbstractUser
from django.db import models
from accounts.models import CustomUser
from django.conf import settings



class Notice(models.Model):
    id = models.IntegerField(primary_key=True)
    user_id = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    notice_type = models.IntegerField() # 1:좋아요, 2:채팅방 3:친구요청 등
    related_id = models.IntegerField() # 댓글id, 채팅방id, 유저id 등
    content = models.TextField()
    is_read = models.BooleanField() # -> 유저가 알림을 읽었는지
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        unique_together = ("notice", "content")

############################################################
# posts (photo)

from django.conf import settings
from django.db import models

class Photo(models.Model):
    # photo_id = models.AutoField(primary_key=True) # Django는 기본적으로 id필드를 자동으로 추가하여 자동으로 값이 증가 -> 만약 사용자 정의로 필드명을 작성하면 해당 내용처럼 필드명을 작성하고 이렇게 되면 장고는 id값을 자동으로 생성하지x
    user_id = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name='photos') # CustomUser.id(o), CustomUser.user_id(x)
    face_chat_id = models.ForeignKey('facechats.FaceChat', on_delete=models.CASCADE, related_name='photos')
    image_url = models.ImageField()
    content = models.CharField(max_length=255, blank=True, null=True)
    count = models.IntegerField(default=0)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)


class Like(models.Model):
    photo_id = models.ForeignKey(Photo, on_delete=models.CASCADE, related_name='likes')
    user_id = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name='likes')
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        unique_together = ('photo', 'user')


class Favorite(models.Model):
    user_id = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name='favorites')
    photo_id = models.ForeignKey(Photo, on_delete=models.CASCADE, related_name='favorites')
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        unique_together = ('user', 'photo')


class Comment(models.Model):
    user_id = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, related_name='comments')
    photo_id = models.ForeignKey(Photo, on_delete=models.CASCADE, related_name='comments')
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    # 대댓글을 위한 self-referential ForeignKey
    parent_id = models.ForeignKey('self', on_delete=models.CASCADE, null=True, blank=True, related_name='replies')

class Tag(models.Model):
    # tag_id = models.AutoField(primary_key=True)
    name = models.CharField(max_length=255, unique=True)

    def __str__(self):
        return self.name
    
class PhotoTag(models.Model):
    photo_id = models.ForeignKey(Photo, on_delete=models.CASCADE, related_name='photo_tags')
    tag_id = models.ForeignKey(Tag, on_delete=models.CASCADE, related_name='photo_tags')

    class Meta:
        unique_together = ('photo', 'tag')


############################################################
############################################################
############################################################
############################################################
############################################################
############################################################
############################################################
############################################################
############################################################
############################################################
############################################################
############################################################
############################################################
