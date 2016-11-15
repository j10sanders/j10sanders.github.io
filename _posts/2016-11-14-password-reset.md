---
layout: post
title:  How to Implement "Password Reset" in an app that doesn't have Flask Security.
category: jekyll 
description: If you want to implement a password recovery/reset function without messing with your entire apps architecture, you may find this helpful.
---


<!--description-->



# Some Background:
### (feel free to skip down to the tutorial)
Last week, I moved my app to a new domain name, and was promptly admonished by my users…  They couldn't remember their passwords!  They had saved their passwords in their browsers, which didn't associate their passwords with the newly changed domain.  So I slapped myself in the forehead, changed the URL back, and decided it was time to finally add the most glaring feature my app currently lacked: the ability for users to change their passwords.

If this was a new app, I would have started by using Flask-Security - a full featured package for handling security features in a Flask application.  It looks pretty slick.  However, my app was working perfectly fine, and I didn't want to change my database models to fit the Flask Security templates, nor did I want to start using WTForms, or the other set of features that Flask-Security uses.  

Instead, I decided to implement my own password recovery feature from the ground up.  I would build this feature into my existing app, instead of changing my app's architecture to fit Flask-Security (or any other security package).

Since I wasn't able to find a good tutorial for implementing password recovery into an app that wasn't already made with Flask-Security, I hope this can be helpful to you if you are in a similar predicament. First, let's think about how password recovery works for most applications (and let's not worry about challenge questions, or extra security features for now).  Then, let's look at the code.






# Behind the scenes - how password recovery features actually work:

It's important to first think about how this feature actually works.  Essentially, what needs to happen is:

1. User enters their registered email address into a field for password reset.
2. A random key is assigned to the user and saved to the database.
3. An email is sent to the user with their key.
4. The user needs to show the application that they received the key.
5. The application verifies the key is the user's (and that it has not expired).
6. The application allows the user to update their password.


# Let's code it up:
I decided to work on the database model first:

	class PWReset(Base):
	    __tablename__ = "pwreset"
	    id = Column(Integer, primary_key=True)
	    reset_key = Column(String(128), unique=True)
	    user_id = Column(Integer, ForeignKey('users.id'), nullable=False)
	    datetime = Column(DateTime(timezone=True), default=datetime.datetime.now)
	    user = relationship(User, lazy='joined')
	    has_activated = Column(Boolean, default=False)

There isn't a whole lot here… so hopefully if any of this doesn't make sense to you, it will by the end of this tutorial.  Since we've added a new database model, a database migration will be necessary. 

Two new html templates need to be added the application: #1 for the user to request a password reset link be sent to their email address, and #2 for the user to update their password after clicking on the URL (which contains the random key).  

Here are some simple templates I made:
### #1 pwresetrq.html:
	{% block content %}
	</br>
	<h1>So you need to reset your password... </h1>
	</br>
	<form role="form" method="POST">
	    <div class="form-group">
	        <label for="email">What email address did you register with?</label>
	        <input type="email" class="form-control" id="email" name="email" placeholder="Email address" required autocomplete="off"> 
	    </div>
	    <button type="submit" class="btn btn-default">Submit</button>
	</form>
	{% endblock %}

### #2 pwreset.html:
	{% block content %}
	</br>
	<h1>Hello, forgetful one... Change your password below:</h1>
	<form role="form" method="POST">
	    <div class="form-group">
	        <label for="password">Password</label>
	        <input type="password" class="form-control" id="password" name="password" placeholder="Password" required autocomplete="off">
	        <input type="password" class="form-control" id="password2" name="password2" placeholder="Password again (to verify)" required autocomplete="off">
	    </div>
	    <button type="submit" class="btn btn-default">Submit</button>
	</form>
	{% endblock %}

Next up, the randomized key needs to be made.  After messing around with a few methods of random string generation, I settled on a stupidly easy one that is built into python.  In an effort to modularize my code, especially in case I change the key generator to a different method later, I made this incredibly tiny keygenerator.py file:

	import uuid
	
	def make_key():
	    return uuid.uuid4()

The uuid module in python was not meant to be a password-reset key generator, but the uuid4 method generates random 32 character strings that look like this: `df917a0f-ae9d-473f-812e-e4f8e72a6088`.  Try calling it a bunch of times - it makes a new random string every time!  If you have 1*10^18 keys generated, there is a 0.14% chance you will have a single duplicate key. So this is safe, super simple, and all that we need.

Now that we have a key, let's assign it to the user in the PWReset table:

	@app.route("/pwresetrq", methods=["POST"])
	def pwresetrq_post():
	    if session.query(User).filter_by(email=request.form["email"]).first():
	           user = session.query(User).filter_by(email=request.form["email"]).one()
	       
	        #check if user already has reset their password, so they will update the current key instead of generating a separate entry in the table.
	        if session.query(PWReset).filter_by(user_id = user.id).first():
	            pwalready = session.query(PWReset).filter_by(user_id = user.id).first()
		#if the key hasn't been used yet, just send the same key.
	            if pwalready.has_activated == False:
	                pwalready.datetime = datetime.now()
	                key = pwalready.reset_key
	            else:    
	                key = keygenerator.make_key()
	                pwalready.reset_key = key
	                pwalready.datetime = datetime.now()
	                pwalready.has_activated = False
	        else:  
	            key = keygenerator.make_key()
	            user_reset = PWReset(reset_key=key, user_id = user.id)
	            session.add(user_reset)
	        session.commit()
	
		##Add Yagmail code here
		#Here is mine:
		''' 
		yag = yagmail.SMTP()
		        contents = ['Please go to this URL to reset your password:', "APP URL HERE" + url_for("pwreset_get",  id = (str(key)))]
		        yag.send('request.form["email"]', 'Reset your password', contents)
		flash(user.name + ", check your email for a link to reset your password.  It expires in a <amount of time here>!", "success")'''
		
	        return redirect(url_for("entries"))
	    else:
	        flash("Your email was never registered.", "danger")
	        return redirect(url_for("pwresetrq_get"))
	
The comments in the code should be helpful for understanding what is happening here: check if the user already has had a pw reset key -- if so, update that row in the database (we don't want the database getting filled with old reset keys).  If the key hasn't been activated yet, reset the datetime (so it doesn't expire) and send the same key.  

[Yagmail](https://github.com/kootenpv/yagmail) is a simple python package for sending email - I recommend checking it out, but you can use whatever email sending service you would like.

Lastly, once the email is sent (with the key included in the URL), we need to give the user the ability to reset their password:

	@app.route("/pwreset/<id>", methods=["GET"])
	def pwreset_get(id):
	    key = id
	    pwresetkey = session.query(PWReset).filter_by(reset_key=id).one()
	    EST = pytz.timezone('US/Eastern')
	    x = datetime.utcnow().replace(tzinfo=pytz.utc).date()- timedelta(days=2)
	    if pwresetkey.has_activated == True:
	        flash("You already reset your password with the URL you are using.  If you need to reset your password again, please make a new request here.", "danger")
	        return redirect(url_for("pwresetrq_get"))
	    if pwresetkey.datetime.replace(tzinfo=pytz.utc).astimezone(EST).date() < x:
	        flash("Your password reset link expired.  Please generate a new one here.", "danger")
	        return redirect(url_for("pwresetrq_get"))
	    return render_template('pwreset.html', id = key)

	@app.route("/pwreset/<id>", methods=["POST"])
	def pwreset_post(id):
	    if request.form["password"] != request.form["password2"]:
	        flash("Your password and password verification didn't match.", "danger")
	        return redirect(url_for("pwreset_get", id = id))
	    if len(request.form["password"]) < 8:
	        flash("Your password needs to be at least 8 characters", "danger")
	        return redirect(url_for("pwreset_get", id = id))
	    user_reset = session.query(PWReset).filter_by(reset_key=id).one()
	       try:
	        session.query(User).filter_by(id = user_reset.user_id).update({'password': generate_password_hash(request.form["password"])})
	        session.commit()
	    except IntegrityError:
	        flash("Something went wrong", "danger")
	        session.rollback()
	        return redirect(url_for("entries"))
	    user_reset.has_activated = True
	    session.commit()
	    flash("Your new password is saved.", "success")
	    return redirect(url_for("entries"))


In this example, I'm giving the user two days to reset their password after a key is generated.  This isn't the best method, since the total amount of time will vary, depending on how late in the day the user requests the password change.  You should feel free to make your own expiration time, and you could also decide to add the expiration time to the database instead of calling it in the python code. 


That should be it!  If you don't have bootstrap "flashes" in your app, you will want to remove all the instance of "flash".  Otherwise, this should work in your Flask app without much fussing around.  Here is the app I implemented it in: https://github.com/j10sanders/crossword.

There are tons of other security features you could add.  What happens if the user forgets their email too?  How about implementing security questions?  How do you stop someone from pegging your email server with password reset requests?  I think these are all interesting questions that are worth pursuing.


