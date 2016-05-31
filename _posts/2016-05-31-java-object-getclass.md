---
layout: post
title: Git常用命令备忘
tags: Git
category: 技术
---

### Object中getClass方法详解

```java
Obejct类有一个getClass()方法：
	返回此 Object 的运行时类。
		返回的 Class 对象是由所表示类的 static synchronized 方法锁定的对象。

		public class Art {
			Art() {
					System.out.println("Art");
							System.out.println(getClass().getName());
								}
								}

								public class Drawing extends Art {
									Drawing() {
											System.out.println("Drawing");
													System.out.println(getClass().getName());
														}
														}

														public class Cartoon extends Drawing{
															Cartoon(){
																	System.out.println("Cartoon");
																			System.out.println(getClass().getName());
																				}
																					
																						public static void main(String[] args) {
																								Cartoon x = new Cartoon();
																										
																												Drawing one = (Drawing)x;
																														Art two = (Art)x;
																																if(one == two){
																																			System.out.println("==");
																																					}else {
																																								System.out.println("!=");
																																										}
																																												System.out.println(x.toString());
																																														System.out.println(one.toString());
																																																System.out.println(two.toString());
																																																	}
																																																	}
																																																	//输出
																																																	Art
																																																	com.cignacmc.knowledge.inheritance.Cartoon
																																																	Drawing
																																																	com.cignacmc.knowledge.inheritance.Cartoon
																																																	Cartoon
																																																	com.cignacmc.knowledge.inheritance.Cartoon
																																																	==
																																																	com.cignacmc.knowledge.inheritance.Cartoon@182f0db
																																																	com.cignacmc.knowledge.inheritance.Cartoon@182f0db
																																																	com.cignacmc.knowledge.inheritance.Cartoon@182f0db

																																																	结论：	当调用getClass()时，返回这个对象真实的Class对象。
																																																			从3个继承对象相等的情况和输出可知，这三个对象有相同的this指针，即内存地址一致。
																																																					而getClass()返回的就是this指针所代表的最真实的Class的对象，也即最上层的子类。


```