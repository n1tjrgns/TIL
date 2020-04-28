```
http://localhost:17201/signup/tma?returl=%2Ftma%2Ftrial
```



```
<div class="signup_group">
					<h4>TMA 가입을 위해 회원정보를 등록하세요.</h4>
				<form th:action="@{/signup/tma}" th:object="${signupForm}" method="post" name="signupForm">
					<input type="hidden" id="depth1" th:field="*{depth1}"/>
					<input type="hidden" id="depth2" th:field="*{depth2}"/>
					<input type="hidden" id="depth3" th:field="*{depth3}"/>
					<input type="hidden" id="depth4" th:field="*{depth4}"/>
					<input type="hidden" id="referer" th:field="*{referer}"/>
					<input type="hidden" id="returl" th:field="*{returl}" th:value="${returl}"/>

					<ul class="tmasu_form">
						<li>
							<h5>아이디<em>*</em></h5>
							<input type="hidden" id="bizType" name="bizType" value="B"/>
							<ul class="input_conts tmaform_id form-id">
								<li>
									<input type="text" id="email" placeholder="이메일을 입력해주세요." title="이메일" onblur="javascript:validateForm();" th:field="*{email}"/>
								</li>
								<li class="txt_info">이메일 형식에 맞게 입력하세요.</li>
								<li class="txt_able">사용 가능한 아이디입니다.</li>
								<li class="txt_error">이미 가입된 아이디입니다.</li>
							</ul>
						</li>
						<li>
							<h5>비밀번호<em>*</em></h5>
							<ul class="input_conts tmaform_pwd form-pwd">
								<li>
									<input type="password" id="password" placeholder="비밀번호를 입력해주세요." title="비밀번호" onblur="javascript:validateForm();" name="password" value="" th:field="*{password}">
								</li>
								<li class="txt_info">숫자,영문,특수문자 등을 혼합하여 8자 이상 입력하세요.</li>
								<li class="txt_able">사용 가능한 비밀번호입니다.</li>
								<li class="txt_error">비밀번호는 숫자,문자,기호 등을 혼합한 8자 이상 입력하세요.</li>
							</ul>
						</li>
						<li>
							<h5>비밀번호 확인<em>*</em></h5>
							<ul class="input_conts tmaform_repwd form-repwd">
								<li>
									<input type="password" id="password_check" placeholder="비밀번호를 다시 입력하세요." title="비밀번호 확인" onblur="javascript:validateForm();" name="repassword" value="" th:field="*{repassword}"/>
								</li>
								<li class="txt_able">비밀번호가 일치합니다.</li>
								<li class="txt_error">다시 입력한 비밀번호가 일치하지 않습니다.</li>
							</ul>
						</li>
						<li class="tmasu_lstcert">
							<h5>본인 인증<em>*</em></h5>
							<ul class="input_conts tmaform_cert form-cert">
								<li>
									<button type="button" onclick="certRequest();" class="btn-lg2 btn-primary">본인 인증</button>
								</li>
							</ul>
							<ul class="input_conts tmaform_cert form-contact" style="display:none;">
								<li>
									<input type="text" id="name" title="담당자" th:field="*{name}" readonly/>
									<input type="hidden" id="certType" th:field="*{certType}"/>
									<input type="hidden" id="certId" th:field="*{certId}"/>
									<input type="text" id="phoneNum" title="연락처" maxlength="40" th:field="*{phoneNum}" readonly/>

									<input type="hidden" id="companyName" th:field="*{companyName}" title="법인명"/>
									<input type="hidden" id="url" th:field="*{url}" title="웹사이트"/>
									<input type="hidden" th:field="*{haveApp}"/>
									<input type="hidden" th:field="*{shoppingMall}"/>
								</li>
							</ul>
						</li>
						<li class="tmasu_lstsign">
							<h5>수신동의</h5>
							<ul class="input_conts tmaform_sign">
								<li>신규서비스 및 기능,프로모션과 혜택에 대해 알려드립니다.</li>
								<!--<li>
									<span>휴대폰</span>
									<input type="checkbox" th:field="*{agPhone}"/>
								</li>
								<li>
									<span>이메일</span>
									<input type="checkbox" th:field="*{agEmail}"/>
								</li>-->
								<li>
									<span>휴대폰</span>
									<div class="choice_chkgray">
										<input type="checkbox" id="choice2" name="choice2" th:field="*{agPhone}">
										<label for="choice2">
											<span></span>
										</label>
									</div>
								</li>
								<li>
									<span>이메일</span>
									<div class="choice_chkgray">
										<input type="checkbox" id="choice3" name="choice3" th:field="*{agEmail}">
										<label for="choice3">
											<span></span>
										</label>
									</div>
								</li>

							</ul>
						</li>
					</ul>
				</form>
			</div>
```

