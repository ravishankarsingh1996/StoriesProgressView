# StoriesProgressView
A WhatsApp, Instagram, social media like story progress view with both image and video play functionality.

This story progress is a modified version of [Yui Kobayashi](https://github.com/shts/StoriesProgressView)

StoriesProgressView
====

Library that shows a horizontal progress like Instagram stories.

[![](https://jitpack.io/v/shts/StoriesProgressView.svg)](https://jitpack.io/#shts/StoriesProgressView)

<img src="https://3.bp.blogspot.com/-EQdxCSXOlP4/XadbQXRU13I/AAAAAAAAWm4/-8Ivn7MEutQScry8SezUh6znfml9Ws0AACK4BGAYYCw/s1600/WhatsApp%2BImage%2B2019-10-16%2Bat%2B11.06.17%2BPM.jpeg" width=200 />

<img src="app/2019_10_16_23_01_06.gif" width=200 /> 

^Modified by me [Ravi Shankar Singh](https://about.me/itsravishankarsingh)

How to Use
----

To see how a StoriesProgressView can be added to your xml layouts, check the sample project.

```xml
    <jp.shts.android.storiesprogressview.StoriesProgressView
        android:id="@+id/stories"
        android:layout_width="match_parent"
        android:layout_height="3dp"
        android:layout_gravity="top"
        android:layout_marginTop="8dp" />
```
Overview

```java
public class MainActivity extends AppCompatActivity implements StoriesProgressView.StoriesListener {

    private static final int PROGRESS_COUNT = 6;
    private StoriesProgressView storiesProgressView;
    private ProgressBar mProgressBar;
    private LinearLayout mVideoViewLayout;
    private int counter = 0;
    private ArrayList<StoriesData> mStoriesList = new ArrayList<>();

    private ArrayList<View> mediaPlayerArrayList = new ArrayList<>();

    long pressTime = 0L;
    long limit = 500L;

    private View.OnTouchListener onTouchListener = new View.OnTouchListener() {
        @Override
        public boolean onTouch(View v, MotionEvent event) {
            switch (event.getAction()) {
                case MotionEvent.ACTION_DOWN:
                    pressTime = System.currentTimeMillis();
                    storiesProgressView.pause();
                    return false;
                case MotionEvent.ACTION_UP:
                    long now = System.currentTimeMillis();
                    storiesProgressView.resume();
                    return limit < now - pressTime;
            }
            return false;
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        getWindow().addFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN);
        setContentView(R.layout.activity_main);
        mProgressBar = findViewById(R.id.progressBar);
        mVideoViewLayout = findViewById(R.id.videoView);
        storiesProgressView = (StoriesProgressView) findViewById(R.id.stories);
        storiesProgressView.setStoriesCount(PROGRESS_COUNT);
        prepareStoriesList();
        storiesProgressView.setStoriesListener(this);
        for (int i = 0; i < mStoriesList.size(); i++) {
            if (mStoriesList.get(i).mimeType.contains("video")) {
                mediaPlayerArrayList.add(getVideoView(i));
            } else if (mStoriesList.get(i).mimeType.contains("image")) {
                mediaPlayerArrayList.add(getImageView(i));
            }
        }

        setStoryView(counter);

        // bind reverse view
        View reverse = findViewById(R.id.reverse);
        reverse.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                storiesProgressView.reverse();
            }
        });
        reverse.setOnTouchListener(onTouchListener);

        // bind skip view
        View skip = findViewById(R.id.skip);
        skip.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                storiesProgressView.skip();
            }
        });
        skip.setOnTouchListener(onTouchListener);
    }

```
I have done some modifications to support videos as well for stories.

```java
 private void setStoryView(final int counter) {
        final View view = (View) mediaPlayerArrayList.get(counter);
        mVideoViewLayout.addView(view);
        if (view instanceof VideoView) {
            final VideoView video = (VideoView) view;
            video.setOnPreparedListener(new MediaPlayer.OnPreparedListener() {
                @Override
                public void onPrepared(MediaPlayer mediaPlayer) {
                    mediaPlayer.setOnInfoListener(new MediaPlayer.OnInfoListener() {
                        @Override
                        public boolean onInfo(MediaPlayer mediaPlayer, int i, int i1) {
                            Log.d("mediaStatus", "onInfo: =============>>>>>>>>>>>>>>>>>>>" + i);
                            switch (i) {
                                case MediaPlayer.MEDIA_INFO_VIDEO_RENDERING_START: {
                                    mProgressBar.setVisibility(View.GONE);
                                    storiesProgressView.resume();
                                    return true;
                                }
                                case MediaPlayer.MEDIA_INFO_BUFFERING_START: {
                                    mProgressBar.setVisibility(View.VISIBLE);
                                    storiesProgressView.pause();
                                    return true;
                                }
                                case MediaPlayer.MEDIA_INFO_BUFFERING_END: {
                                    mProgressBar.setVisibility(View.VISIBLE);
                                    storiesProgressView.pause();
                                    return true;

                                }
                                case MediaPlayer.MEDIA_ERROR_TIMED_OUT: {
                                    mProgressBar.setVisibility(View.VISIBLE);
                                    storiesProgressView.pause();
                                    return true;
                                }

                                case MediaPlayer.MEDIA_INFO_AUDIO_NOT_PLAYING: {
                                    mProgressBar.setVisibility(View.VISIBLE);
                                    storiesProgressView.pause();
                                    return true;
                                }
                            }
                            return false;
                        }
                    });
                    video.start();
                    mProgressBar.setVisibility(View.GONE);
                    storiesProgressView.setStoryDuration(mediaPlayer.getDuration());
                    storiesProgressView.startStories(counter);
                }
            });
        } else if (view instanceof ImageView) {
            final ImageView image = (ImageView) view;
            mProgressBar.setVisibility(View.GONE);
            Glide.with(this).load(mStoriesList.get(counter).mediaUrl).addListener(new RequestListener<Drawable>() {
                @Override
                public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
                    Toast.makeText(MainActivity.this, "Failed to load media...", Toast.LENGTH_SHORT).show();
                    mProgressBar.setVisibility(View.GONE);
                    return false;
                }

                @Override
                public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
                    mProgressBar.setVisibility(View.GONE);
                    storiesProgressView.setStoryDuration(5000);
                    storiesProgressView.startStories(counter);
                    return false;
                }
            }).into(image);
        }
    }

```

Skip and Reverse story
---

```java
  storiesProgressView.skip();
  storiesProgressView.reverse();
```

Pause and Resume story
---


```java
  storiesProgressView.pause();
  storiesProgressView.resume();
```


Install
---

Add it in your root build.gradle at the end of repositories:

```groovy
allprojects {
    repositories {
        ...
        maven { url "https://jitpack.io" }
    }
}

```

Add the dependency

```
dependencies {
    implementation 'com.github.shts:StoriesProgressView:3.0.0'
}

```

License
---

```
Copyright (C) 2019 Ravi Shankar Singh

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
