# Test
//测试
package com.example.xueba;

import java.util.ArrayList;
import java.util.Collections;
import java.util.Random;

import android.app.Activity;
import android.app.AlertDialog;
import android.content.DialogInterface;
import android.content.DialogInterface.OnClickListener;
import android.os.Bundle;
import android.os.Handler;
import android.view.View;
import android.view.Window;
import android.view.animation.Animation;
import android.view.animation.TranslateAnimation;
import android.widget.Button;
import android.widget.TextView;

public class PlayActivity extends Activity {
	
	private ArrayList<Question> questionList;//问题列表
	private int questionIndex = 0;//当前问题的索引
	Random random = new Random();
	private int rightCount = 0;//答对题目的数量
	
	private static final int MAX_COUNT = 5;//一次答题的数量

	//控件
	TextView titleTextView, tipTextView, qurstionTextView;
	Button[] buttons = new Button[4];//备选答案
	int[] ids = {R.id.button_A, R.id.button_B, R.id.button_C, R.id.button_D};
	

	private String correctAnswer;//当前题目的正确答案
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		
		requestWindowFeature(Window.FEATURE_NO_TITLE);//隐藏标题(要写在SetContentView前面)
		
		setContentView(R.layout.activity_play);
		titleTextView = (TextView) findViewById(R.id.textView_Title);
		tipTextView = (TextView) findViewById(R.id.textView_Tip);
		qurstionTextView = (TextView) findViewById(R.id.textView_Question);
		
		for (int i = 0; i < ids.length; i++) {
			buttons[i] = (Button) findViewById(ids[i]);
		}
		
		//获取启动该Activity的intent传递的参数
		String subject = getIntent().getStringExtra("subject");
		System.out.println("选择的主题是:" + subject);
		titleTextView.setText(subject);//显示主题
		
		MyApplication app = (MyApplication) getApplication();
		questionList = app.getQuestionListBySubject(subject);
		Collections.shuffle(questionList);//洗牌
		
		showQuestion();
	}
	
	/**
	 * 显示题目
	 */
	void showQuestion() {
		//1.获取一个Question
		Question question = questionList.get(questionIndex++);//随机取一个题目
		questionIndex %= questionList.size();//限定范围
		tipTextView.setText("第" + questionIndex + "/" + MAX_COUNT + "题");
		
		//2.显示到界面
		qurstionTextView.setText(question.getQuestion());
		buttons[0].setText(question.getAnswer1());
		buttons[1].setText(question.getAnswer2());
		buttons[2].setText(question.getAnswer3());
		buttons[3].setText(question.getAnswer4());
		
		//将正确答案随机放到4个选项上面
		int j = random.nextInt(4);
		correctAnswer = question.getAnswer1();
		buttons[0].setText(buttons[j].getText());
		buttons[j].setText(correctAnswer);
		
		//执行按钮从右边飞入动画
		startButtonAnimation();
		
	}
	
	/**
	 * 执行按钮从右边飞入动画
	 */
	private void startButtonAnimation() {
		for (int i = 0; i < buttons.length; i++) {
			TranslateAnimation animation = new TranslateAnimation(Animation.RELATIVE_TO_PARENT, 1.0f, Animation.RELATIVE_TO_PARENT, 0, Animation.RELATIVE_TO_PARENT, 0, Animation.RELATIVE_TO_PARENT, 0);
			animation.setDuration(666 + 150 * i);
			
			buttons[i].startAnimation(animation);
		}
		
	}

	/**
	 * 备选答案被点击
	 * @param v
	 */
	public void onClick(View v) {
		
		Button button = (Button) v;
		String text = button.getText().toString();
		if (text.equals(correctAnswer)) {
			rightCount++;//记录答对题目加一
			//正确
			switch (v.getId()) {
			case R.id.button_A: button.setBackgroundResource(R.drawable.answerbr_a);break;
			case R.id.button_B: button.setBackgroundResource(R.drawable.answerbr_b);break;
			case R.id.button_C: button.setBackgroundResource(R.drawable.answerbr_c);break;
			case R.id.button_D: button.setBackgroundResource(R.drawable.answerbr_d);break;
			}
		}else {
			//错误
			switch (v.getId()) {
			case R.id.button_A: button.setBackgroundResource(R.drawable.answerx_a);break;
			case R.id.button_B: button.setBackgroundResource(R.drawable.answerx_b);break;
			case R.id.button_C: button.setBackgroundResource(R.drawable.answerx_c);break;
			case R.id.button_D: button.setBackgroundResource(R.drawable.answerx_d);break;
			}
		}
		
		//延时1000毫秒,跳下一题(重点Handler)
		new Handler().postDelayed(new Runnable() {
			@Override
			public void run() {
				
				if (questionIndex >= 5) {
					//弹出对话框,提示答题完毕，显示成绩
					AlertDialog.Builder builder = new AlertDialog.Builder(PlayActivity.this);
					builder.setTitle("答题结束");
					builder.setMessage("答对:" + rightCount + "\n答错:" + (MAX_COUNT - rightCount));
					builder.setPositiveButton("再战", new OnClickListener() {
						
						@Override
						public void onClick(DialogInterface arg0, int arg1) {
							questionIndex = rightCount = 0;
							Collections.shuffle(questionList);
							resetButtons();//还原按钮背景
							showQuestion();
						}
					});
					
					builder.setNegativeButton("拜拜", new OnClickListener() {
						
						@Override
						public void onClick(DialogInterface arg0, int arg1) {
							PlayActivity.this.finish();
						}
					});
					builder.show();
					
				}else {
					resetButtons();//还原按钮背景
					showQuestion();
				}
				
			}
		}, 1000);
		
		
	}

	/**
	 * 还原按钮背景
	 */
	private void resetButtons() {
		buttons[0].setBackgroundResource(R.drawable.selector_button_a);
		buttons[1].setBackgroundResource(R.drawable.selector_button_a);
		buttons[2].setBackgroundResource(R.drawable.selector_button_a);
		buttons[3].setBackgroundResource(R.drawable.selector_button_a);
	}
	

}
