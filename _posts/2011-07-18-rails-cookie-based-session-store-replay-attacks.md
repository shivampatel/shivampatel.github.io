---
layout: post
title:  "Rails applications using cookie based session store for session management are vulnerable to Replay Attacks"
categories: jekyll update
---

<div class="content" style="margin-top: 15px;">
        <p>Rails3 uses Cookie Store as the default for session management . This is a significant (and interesting) change that has a couple implications on the security of web application that rely on cookies for session storage. People <a href="http://www.ruby-forum.com/topic/102147" target="_blank">have already illustrated</a> how replay attacks can subvert your web applications.</p>
        <p>This article demonstrates a replay attack where – in certain cases – your login based application’s logic can be subverted to bypass the authentication and directly impersonate an user (whose cookie you’ve managed to sniff/steal).</p>
        <p>This post will:</p>
        <ul>
          <li>Illustrate how sessions are stored in cookies instead of server side storage.</li>
          <li>Demonstrate a common mechanism in web applications to identify if the user is authenticated.</li>
          <li>Show how can a stolen cookie can be replayed to impersonate a valid user (even after the real user logs off).</li>
        </ul>
        <p>In cookie based session stores, all the session data is stored in a cookie as a hash map of key:value pairs. Everything that you set in a session is serialized into a stream of bytes – base64 encoded – and set as a browser cookie. Hence, all of your session variables actually end up being stored on client side. From then on, <strong>your browser sends all your session data to the server in each single request you make to the website.</strong></p>
        <p>However there is a security mechanism that prevents from any user from tampering the session data stored as cookie – and that is the <a href="http://en.wikipedia.org/wiki/HMAC" target="_blank">HMAC</a> of the session byte stream calculated from a server side secret. So here is how it works:</p>
        <ul>
          <li>When you do a session[:valid] = “true” in a rails application, the application converts it into a hash, serializes it into a byte stream and calculates a HMAC of this stream.</li>
          <li>This HMAC is appended along with the session object in the cookie with a ‘- -’ delimeter.</li>
          <li>The browser sends this cookie (session object–HMAC) with every request to server.</li>
          <li>For every request, the server <strong>calculates </strong>the HMAC of the session hash that browser sent in the cookie and compares it with the accompanying HMAC in the cookie. If they don’t match, the session hash (or the HMAC) has been modified on client side – the session data will be ignored in this case.</li>
        </ul>
        <p>Now lets see a common way how sessions are managed in many web applications.<br>
          Suppose your web applications’ authentication routine looks like this:</p>

     <pre class="prettyprint lang-rb">
def verify
  if params[:username]=='demo' && params[:password]=='demo'
    flash[:message] = "Login Successful! Welcome to the member area"
    session[:valid] = "true"
    session[:name] = "Shivam Patel"
    session[:email] = "shivam@cmu.edu"
    session[:password] = "commack@782"
    session[:credits] = "780"
    redirect_to '/members/profile'
  else
    flash[:message] = "Your username/password is incorrect. Did you try demo/demo :)"
     redirect_to '/auth/login'
  end
end
    </pre>


      <p>The code above checks for the valid credentials, and sets some session variables once the user is authenticated. A common technique employed for session management is to set a special session variable once an user is authenticated. Thereafter for each request by the browser, a simple check is performed to see if that session variable is set or not. If its set, the requested ‘member only’ page is displayed else an error is displayed asking user to login to access that page. This value is cleared when user presses logout. In our code above, session[:valid] serves the purpose of the ‘flag’ to denote a authenticated user.</p>
        <p>Consecutively, our ‘members only’ pages will have something like this defined in their controller:</p>
<pre class="prettyprint lang-rb">
class MembersController < ApplicationController
before_filter :check_login_status
</pre>
        <p>Where check_login_status is a method which will be evaluated before every action in this controller. The check_login_status will look something like:</p>
<pre class="prettyprint">
class ApplicationController < ActionController::Base
  protect_from_forgery
  def check_login_status
    if session[:valid] != "true"
      flash[:message] = "That is a 'member only' page, please login with valid credentials to access it"
      redirect_to "/auth/login"
    end
  end
end
</pre>

        <p>I’ve put up a demo application at <a href="http://replayattack.mocktest.net" target="_blank">http://replayattack.mocktest.net</a> where you can play with a simple login based application which uses cookie based session store. You can try the replay attack (described below) on this application.</p>
        <p>I use a simple Firefox extension – <a href="https://addons.mozilla.org/en-US/firefox/addon/live-http-headers/" target="_blank">Live HTTP headers</a> for evaluating and replaying the cookie. So, lets login at the application (demo/demo) and lets see what cookie we got. The headers for ‘/members/profile’ page sets this cookie.</p>
<pre class="prettyprint">
Set-Cookie: _rails_demo_session=BAh7DUkiD3Nlc3Npb25faWQGOgZFRiIlZTBhZWRjZGE3MGE5MTk5MGMyNTllMDU1ZWRkMGRkMDdJIhBfY3NyZl90b2tlbgY7AEZJIjFicFl4aUpuSURIWkVhaE8rTVpuTDgrUWNvdzlwRnYzVWJhcEYwazBDRzM0PQY7AEZJIgp2YWxpZAY7AEZJIgl0cnVlBjsARkkiCW5hbWUGOwBGSSIRU2hpdmFtIFBhdGVsBjsARkkiCmVtYWlsBjsARkkiE3NoaXZhbUBjbXUuZWR1BjsARkkiDXBhc3N3b3JkBjsARkkiEGNvbW1hY2tANzgyBjsARkkiDGNyZWRpdHMGOwBGSSIINzgwBjsARkkiCmZsYXNoBjsARklDOiVBY3Rpb25EaXNwYXRjaDo6Rmxhc2g6OkZsYXNoSGFzaHsGOgxtZXNzYWdlSSIxTG9naW4gU3VjY2Vzc2Z1bCEgV2VsY29tZSB0byB0aGUgbWVtYmVyIGFyZWEGOwBGBjoKQHVzZWRvOghTZXQGOgpAaGFzaHsA–8033ab5323d8adfb80a32fa448a41a586138ee4a; path=/; HttpOnly
</pre>

        <p>You can now see two distinct parts of this cookie: The base64 encoded session data followed by the delimiter – - followed by the HMAC (8033ab5323d8adfb80a32fa448a41a586138ee4a). If you use any <a href="http://www.opinionatedgeek.com/dotnet/tools/base64decode/" target="_blank">base64 decoder</a> and try to decode the session data part of the cookie (BAh7…. aHsA), you’ll see the serialized representation of the session hash:</p>
<pre class="prettyprint lang-js">
{
I"session_id:EF"%e0aedcda70a91990c259e055edd0dd07I"_csrf_token;FI"1bpYxiJnIDHZEahO+MZnL8+Qcow9pFv3UbapF0k0CG34=;FI"
valid;FI"	true;FI"	name;FI"Shivam Patel;FI"
email;FI"shivam@cmu.edu;FI"
password;FI"commack@782;FI"credits;FI"780;FI"
flash;FIC:%ActionDispatch::Flash::FlashHash{:messageI"1Login Successful! Welcome to the member area;F:
@usedo:Set:
@hash{
</pre>
        <p>We can see the session variables highlighted in this semi-readable text. Can you change the value of ‘credits’ in the cookie and fool the server to accept that. Fortunately not, because the HMAC won’t match. <strong>But you can always store this cookie to replay it later.</strong> And here lies the problem.</p>
        <p>Now we’ll logout from the application by clicking the ‘Logout’ link on the web application. The logout code is:</p>
<pre class="prettyprint">
def logout
  reset_session
  flash[:message] = "You have been successfully logged out !"
  redirect_to "/auth/login"
end
</pre>
        <p>This resets the session and generates a new session id. If you inspect the new cookie (post logout) in Live HTTP headers,
          you’ll no longer find those session variables.</p>
        <div id="attachment_112" style="width: 462px; float: right;">
            <img title="Locate an old GET request in Live HTTP header and replay it" src=<%= image_path "blog/rails_cookie_store_session_management_replay_attack1.png"%> alt="" width="452" height="493">
          <p class="wp-caption-text">Locate an old GET request in Live HTTP header and replay it</p></div>
        <p>But what if you just replay that old cookie. In Live HTTP headers, all you need to do it to scroll up to
          reach the request which was made to a page before you clicked logout and “Replay it”.</p>
        <p>As soon as you click that, BINGO !!! – you are logged in as a valid user.</p>
        <p>This happened because the session[:valid] flag we are checking on server side to see if the user has already
          authenticated to the application is set in this cookie which obviously is of a time before users clicked
          ‘Logout’. This is a very serious issue. You need to worry if your applications’ login design is similar to
          one described here.<br>
          This will allow people to sniff your users cookie if they are on a public wireless (airports, coffee shops)
          and replay it to gain access to your application <strong>even when the real user has logged off the
          application</strong>.</p>
        <p>You are safe from such an attack if:</p>
        <ul>
          <li>You use SSL on your website. However, remember that SSL should be enabled for the entire site rather
            than for the login page only. For sites using SSL, there is no way an attacker can sniff plain text cookies
            and try to read/replay them.
            <ul>
              <li>Also make sure that you set the ‘secure’ cookie flag so that the cookie is never inadvertently
                transmitted on a http channel</li>
            </ul>
          </li>
          <li>You are not using cookie store to manage your sessions.</li>
        </ul>
        <p>If you want to quickly migrate from cookie based session store to database based sessions, its actually very
          easy. However you may want to think about the performance impact it will have based on the way you’ve
          deployed your application.</p>
        <p>In case you want to switch to database backed sessions,
          <a href="http://stackoverflow.com/questions/2588241/rails-sessions-current-practices-especially-for-rails-3" target="_blank">
            here is how to</a>.</p>
        <p>Hope this post helped you understand how cookie based sessions work and why and when you should not use
          CookieStore. Have fun doing the replays at <a href="http://replayattack.mocktest.net" target="_blank">http://replayattack.mocktest.net</a>.
          Don’t mount other nefarious attacks on it please.</p>

  </div> <!-- content div -->
