---
layout: page
title: Tags
---
<div class="tag_posts">
</div>

function showPosts(tagContent) {
    var tag = tags.filter(function(e) { return e.tag == tagContent; });
    if (tag.length == 0) {
        return;
    }
    tag = tag[0];
    var $ul = $('<ul></ul>').append(tag.posts.map(function(post) {
        var $li = $('<li></li>');
        $li.html(post.date + ' - ' + '<a href="' + post.url + '">' + post.title + '</a>');
        return $li;
    }));
    $('.tag_posts').slideUp('normal', function(e) {
        $('.tag_posts').empty().append($('<div class="tag_title">' + tagContent + '</div>')).append($ul).slideDown('normal');
    });
}

if (location.hash != '') {
    showPosts(location.hash.substring(1));
}
