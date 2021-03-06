ki4so服务器配置详细说明
=====

# 基本场景介绍 #

当您决定使用ki4so作为贵公司的sso服务之后，那么必然需要与自己的用户信息系统进行集成，因为ki4so本身并没有包含用户信息管理系统，因为ki4so要保持自己小巧的身段，以后也不会将用户管理包含在ki4so里面。那么本文主要是介绍如何与自身的用户信息管理系统如何进行集成。使用自己开发的用户信息管理系统实现用户登录。


# 实现认证处理器 #

需要集成企业内部的用户信息管理系统，需要实现自己的认证管理器类，最重要的一个方法就是识别用户名和密码是否匹配。

对于一般的场景，即通过用户名和密码登录认证的场景，我们可以参考ki4so-core中默认的一个测试用的认证处理器实现类的方法实现自己的认证处理器类，该类全名是：com.github.ebnew.ki4so.core.authentication.handlers.SimpleTestUsernamePasswordAuthenticationHandler,它位于项目ki4so-core中。它的代码很简单，如下所示。

    public final class SimpleTestUsernamePasswordAuthenticationHandler extends
		AbstractUsernamePasswordAuthenticationHandler {

	public SimpleTestUsernamePasswordAuthenticationHandler() {

	}

	@Override
	public boolean authenticateUsernamePasswordInternal(
			final UsernamePasswordCredential credential) {
		final String username = credential.getUsername();
		final String password = credential.getPassword();

		if (StringUtils.hasText(username) && StringUtils.hasText(password)
				&& username.equals(getPasswordEncoder().encode(password))) {
			return true;
		}
		throw UsernameOrPasswordInvalidException.INSTANCE;
	}
}


你自己的认证处理器也可以继承自类AbstractUsernamePasswordAuthenticationHandler，然后实现你自身的认证逻辑，从数据库中查询用户名对应的密码，或者通过远程方法调用查询对应的密码，密码一般会通过加密算法进行加密，则你需要实现合适的加密算法。

如果上述结构不符合您的具体使用场景，则可以通过实现更高参差的接口来实现认证处理器，可以从AbstractUsernamePasswordAuthenticationHandler网上追溯更高层次的接口来实现你的处理器，层次越高越抽象，灵活性也越高，应该能够绝大多数使用场景的。

# 实现密码加密器 #
出于安全考虑，对于密码我们一般都会通过加密算法加密之后进行存储，那么在比较密码的时候，我们一般要先进行加密之后再进行比较，那么ki4so默认的实现是使用明文算法进行比较，因此你需要实现自己的加密器以符合自身的需求。


如果是MD5或者SHA1加密的话则在ki4so-core中有一个默认的实现类，即类：com.github.ebnew.ki4so.core.authentication.handlers.DefaultPasswordEncoder,如果该类能够满足您的需求，则直接配置使用该类即可。

如果不能满足您的需求，则可以参考该类实现接口com.github.ebnew.ki4so.core.authentication.handlers.PasswordEncoder即可，该接口只有一个方法需要实现，非常简单，只需要实现该方法即可。

        /**
     * Method that actually performs the transformation of the plaintext
     * password into the encrypted password.
     * 
     * @param password the password to translate
     * @return the transformed version of the password
     */
    String encode(String password);

该方法有详细的注释说明，值需要将密码进行加密换算即可，输出是加密后的密文。


# 配置认证处理器 #

对于了解j2EE开发的同学来说，应该对于Spring不会陌生的，同样为了提高ki4so的代码灵活性和通用性，ki4so也使用了Spring的IOC来配置bean，对于自己实现的认证处理器